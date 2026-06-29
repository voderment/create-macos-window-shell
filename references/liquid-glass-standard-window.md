# Liquid Glass Standard Window

Use this recipe for a normal macOS app/document window that keeps native close/minimize/zoom behavior, resize behavior, rounded unified chrome, and system shadow while rendering a bare Liquid Glass content container.

This is a shell recipe, not a feature UI recipe. Start with an empty content area such as `EmptyView()` or a caller-supplied generic `Content`; do not include drop targets, dashed cards, buttons, onboarding copy, or sample controls.

## Shape

- Standard `NSWindow`, not borderless.
- Style mask: `.titled`, `.closable`, `.miniaturizable`, `.resizable`, `.fullSizeContentView`.
- Transparent titlebar with hidden title and no separator.
- Empty unified `NSToolbar` to trigger native rounded unified chrome.
- Clear, non-opaque window background.
- Native `NSGlassEffectView` backdrop on macOS 26, with `NSVisualEffectView` fallback.
- Uniform dark scrim over the glass when a dark window is desired.
- Optional traffic-light position controller when the design needs custom inset.

## Window Configuration

Perform window setup on the main actor:

```swift
@MainActor
enum GlassWindowFactory {
    static func makeWindow<Content: View>(
        size: NSSize = NSSize(width: 980, height: 620),
        minSize: NSSize = NSSize(width: 640, height: 420),
        @ViewBuilder content: () -> Content
    ) -> NSWindow {
        let window = NSWindow(
            contentRect: NSRect(origin: .zero, size: size),
            styleMask: [
                .titled,
                .closable,
                .miniaturizable,
                .resizable,
                .fullSizeContentView
            ],
            backing: .buffered,
            defer: false
        )

        window.title = ""
        window.titleVisibility = .hidden
        window.titlebarAppearsTransparent = true
        window.titlebarSeparatorStyle = .none
        window.toolbarStyle = .unified

        let toolbar = NSToolbar(identifier: "GlassWindowToolbar")
        toolbar.displayMode = .iconOnly
        toolbar.allowsUserCustomization = false
        toolbar.autosavesConfiguration = false
        if #available(macOS 15.0, *) {
            toolbar.allowsDisplayModeCustomization = false
        }
        window.toolbar = toolbar

        window.isOpaque = false
        window.backgroundColor = .clear
        window.hasShadow = true
        window.isMovableByWindowBackground = true
        window.minSize = minSize
        window.center()

        let rootView = GlassWindowShell {
            content()
        }
        window.contentView = NSHostingView(rootView: rootView)
        return window
    }
}
```

Use `window.isMovableByWindowBackground = true` only while the shell is mostly empty. For complex content, set it to `false` and add a dedicated drag area.

## SwiftUI Shell

Keep the shell generic and bare:

```swift
struct GlassWindowShell<Content: View>: View {
    @ViewBuilder var content: Content

    var body: some View {
        let shape = RoundedRectangle(cornerRadius: 30, style: .continuous)

        ZStack {
            GlassWindowSurface(shape: shape)
                .allowsHitTesting(false)

            content
                .padding(.top, 54)
                .padding(.horizontal, 32)
                .padding(.bottom, 32)
        }
        .clipShape(shape)
        .overlay {
            shape.strokeBorder(Color.white.opacity(0.14), lineWidth: 1)
        }
        .ignoresSafeArea()
    }
}

struct GlassWindowSurface<S: InsettableShape>: View {
    var shape: S

    var body: some View {
        ZStack {
            NativeGlassBackdropView()

            shape.fill(Color.black.opacity(0.30))
        }
        .clipShape(shape)
    }
}
```

Tune only the uniform scrim opacity first. Good dark-window values are usually `0.22...0.34`. Avoid directional gradients for the base shell.

## Native Backdrop

Use a pass-through AppKit view so the backdrop never steals input:

```swift
@MainActor
final class PassthroughGlassContainerView: NSView {
    override func hitTest(_ point: NSPoint) -> NSView? {
        nil
    }
}

struct NativeGlassBackdropView: NSViewRepresentable {
    func makeNSView(context: Context) -> NSView {
        let container = PassthroughGlassContainerView()
        container.wantsLayer = true

        let glass = makeGlassView()
        glass.frame = container.bounds
        glass.autoresizingMask = [.width, .height]
        container.addSubview(glass)

        return container
    }

    func updateNSView(_ view: NSView, context: Context) {
        view.subviews.first?.frame = view.bounds
    }

    private func makeGlassView() -> NSView {
        if
            #available(macOS 26.0, *),
            let glassType = NSClassFromString("NSGlassEffectView") as? NSView.Type
        {
            let view = glassType.init(frame: .zero)
            view.wantsLayer = true
            return view
        }

        let view = NSVisualEffectView()
        view.material = .underWindowBackground
        view.blendingMode = .behindWindow
        view.state = .active
        view.wantsLayer = true
        return view
    }
}
```

Use `NSGlassEffectView` as the full-window backdrop. Use SwiftUI `.glassEffect` for smaller controls or panels inside the content, not for the primary window background unless the target design specifically calls for component-style glass.

## Traffic Lights

Prefer the system position when it works. If a design needs deeper inset, keep a retained controller and update on resize:

```swift
@MainActor
final class WindowChromeController: NSObject, NSWindowDelegate {
    private weak var window: NSWindow?
    private let leftInset: CGFloat
    private let topInset: CGFloat
    private let miniaturizeOffset: CGFloat
    private let zoomOffset: CGFloat

    init(window: NSWindow, leftInset: CGFloat = 34, topInset: CGFloat = 30) {
        self.window = window
        self.leftInset = leftInset
        self.topInset = topInset

        let close = window.standardWindowButton(.closeButton)
        let miniaturize = window.standardWindowButton(.miniaturizeButton)
        let zoom = window.standardWindowButton(.zoomButton)
        self.miniaturizeOffset = (miniaturize?.frame.minX ?? 0) - (close?.frame.minX ?? 0)
        self.zoomOffset = (zoom?.frame.minX ?? 0) - (close?.frame.minX ?? 0)

        super.init()
        window.delegate = self
        positionTrafficLights()
        DispatchQueue.main.async { [weak self] in self?.positionTrafficLights() }
    }

    func windowDidResize(_ notification: Notification) {
        positionTrafficLights()
    }

    private func positionTrafficLights() {
        guard
            let window,
            let close = window.standardWindowButton(.closeButton),
            let miniaturize = window.standardWindowButton(.miniaturizeButton),
            let zoom = window.standardWindowButton(.zoomButton),
            let buttonContainer = close.superview
        else { return }

        let y = buttonContainer.bounds.height - topInset - close.frame.height
        close.setFrameOrigin(NSPoint(x: leftInset, y: y))
        miniaturize.setFrameOrigin(NSPoint(x: leftInset + miniaturizeOffset, y: y))
        zoom.setFrameOrigin(NSPoint(x: leftInset + zoomOffset, y: y))
    }
}
```

Store this controller strongly for the lifetime of the window.

## Electron Adaptation

For Electron projects, adapt the same layers:

- `BrowserWindow`: `titleBarStyle: "hidden"`, `transparent: true`, `backgroundColor: "#00000000"`, `roundedCorners: true`, and a tuned `trafficLightPosition`.
- Native addon: get the native window handle, configure the underlying `NSWindow` with transparent titlebar, full-size content view, empty unified toolbar, clear non-opaque background, and movable background.
- Insert a pass-through `NSGlassEffectView` or `NSVisualEffectView` below web contents.
- CSS: keep `body` transparent; mark drag regions and no-drag controls explicitly.

The reference implementation that motivated this recipe is `AlexandrosGounis/pdfx`, especially its native `glass.mm` window backdrop pattern.

## Validation

- Build with the project command.
- Verify there is no demo content in the shell.
- Search shell files for `LinearGradient`, `RadialGradient`, `Drop files`, `Browse`, or dashed placeholder borders when a bare window was requested.
- Confirm traffic lights are real system buttons and remain clickable.
- Confirm the glass backdrop does not intercept mouse events.
- Confirm resize keeps the backdrop filling the window.
