# WKWebView With PopUp
WKWebView handling pop-up windows

## Property
If there is a pop-up window, use the pop-up window. If there is no pop-up window, use the default web view as the current webView.
```swift
var popupWebViews = [WKWebView]()
var currentWebView: WKWebView {
    return popupWebViews.isEmpty ? webView : popupWebViews.last!
}
```

## Settings
### 1. configure javaScript
```swift
if #available(iOS 14, *) {
    let preferences = WKWebpagePreferences()
    preferences.allowsContentJavaScript = true
    webView.configuration.defaultWebpagePreferences = preferences
}
else {
    webView.configuration.preferences.javaScriptEnabled = true
    /// user interaction없이 윈도우 창을 열 수 있는지 여부를 나타냄. iOS에서는 기본값이 false이다.
    webView.configuration.preferences.javaScriptCanOpenWindowsAutomatically = true
}
```

### 2. back and forward button
```swift
func configureBackwardAndForwardButton() {
    /// webView가 뒤로 이동할 수 있거나 팝업창이 있는 경우 뒤로가기 활성화
    backwardButton.isEnabled = currentWebView.canGoBack || popupWebViews.count > 0
    /// webView가 앞으로 이동할 수 있는 경우 활성화
    forwardButton.isEnabled = currentWebView.canGoForward
}
```

### 3. backward button action
```swift
@IBAction func tapBackwardButton(_ sender: UIBarButtonItem) {
    if currentWebView.canGoBack {
        currentWebView.goBack()
        return
    }

    if popupWebViews.count > 0 {
        let last = popupWebViews.popLast()
        last?.removeFromSuperview()
        /// webView 로드가 완료되면 버튼을 재설정하기는 하지만
        /// 팝업창을 뒤로가기할 경우(닫을 경우) 로드 완료 delegate가 호출되지 않기 때문에 이곳에서 버튼을 재설정한다.
        configureBackwardAndForwardButton()
    }
}
```

### 4. create new pop-up
```swift
func createNewWebView(_ config: WKWebViewConfiguration) -> WKWebView {
    let newWebView = WKWebView(frame: webView.frame,
                           configuration: config)
    newWebView.navigationDelegate = self
    newWebView.uiDelegate = self
    newWebView.allowsBackForwardNavigationGestures = true
    view.addSubview(newWebView)
    popupWebViews.append(newWebView)
    return newWebView
}
```

## Delegate
```swift
func webView(_ webView: WKWebView, createWebViewWith configuration: WKWebViewConfiguration, for navigationAction: WKNavigationAction, windowFeatures: WKWindowFeatures) -> WKWebView? {
    /// 팝업창(새 탭) 처리
    print("--->[createWebViewWith configuration]")
    return createNewWebView(configuration)
}
```

