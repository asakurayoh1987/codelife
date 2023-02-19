#### Android WebView 选择文件操作

实现方法，利用`WebChromeClient` 的`onShowFileChooser` 实现文件选择功能

```java
public class H5FileHelper {

    ValueCallback<Uri[]> mUploadMessage;
    public final static int FILECHOOSER_RESULTCODE = 2;

    public H5FileHelper(WebView webView) {
        setWebChromeClient(webView);
    }

    private void setWebChromeClient(WebView webView) {
        webView.setWebChromeClient(new WebChromeClient() {
            @Override
            public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
                mUploadMessage = filePathCallback;
                Intent contentSelectionIntent = new Intent(Intent.ACTION_GET_CONTENT);
                contentSelectionIntent.addCategory(Intent.CATEGORY_OPENABLE);
                contentSelectionIntent.setType("image/*");
                Intent chooserIntent = new Intent(Intent.ACTION_CHOOSER);
                chooserIntent.putExtra(Intent.EXTRA_INTENT, contentSelectionIntent);
                chooserIntent.putExtra(Intent.EXTRA_TITLE, "Image Chooser");
                ((Activity) webView.getContext()).startActivityForResult(chooserIntent, FILECHOOSER_RESULTCODE);
                return true;
            }
        });
    }

    /**
     * WebView对应的Activity的onActivityResult回调回来在此处处理
     *
     * @param requestCode
     * @param resultCode
     * @param intent
     */
    public void onActivityResult(int requestCode, int resultCode, Intent intent) {
        if (requestCode == FILECHOOSER_RESULTCODE) {
            if (null == mUploadMessage)
                return;
            Uri result = (intent == null || resultCode != Activity.RESULT_OK) ? null : intent.getData();
            //无论是否选择都必须回调, 否则下次点击不会回调onShowFileChooser
            if (result != null) {
                mUploadMessage.onReceiveValue(new Uri[]{result});
            } else {
                mUploadMessage.onReceiveValue(new Uri[]{});
            }
            mUploadMessage = null;
        }
    }
}

```

