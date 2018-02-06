# 支付标准sdk_demo

## 核心代码
### 支付核心代码
```java
final String orderInfo = "服务端返回";

Runnable payRunnable = new Runnable() {

	@Override
	public void run() {
		PayTask alipay = new PayTask(PayDemoActivity.this);
		Map<String, String> result = alipay.payV2(orderInfo, true);
		Log.i("msp", result.toString());
		
		Message msg = new Message();
		msg.what = SDK_PAY_FLAG;
		msg.obj = result;
		mHandler.sendMessage(msg);
	}
};

Thread payThread = new Thread(payRunnable);
payThread.start();
```
**orderInfo由服务端加签完成后返回，客户端只需要使用**
### 处理返回结果
```java
@SuppressLint("HandlerLeak")
private Handler mHandler = new Handler() {
	@SuppressWarnings("unused")
	public void handleMessage(Message msg) {
		switch (msg.what) {
		case SDK_PAY_FLAG: {
			@SuppressWarnings("unchecked")
			PayResult payResult = new PayResult((Map<String, String>) msg.obj);
			/**
			 对于支付结果，请商户依赖服务端的异步通知结果。同步通知结果，仅作为支付结束的通知。
			 */
			String resultInfo = payResult.getResult();// 同步返回需要验证的信息
			String resultStatus = payResult.getResultStatus();
			// 判断resultStatus 为9000则代表支付成功
			if (TextUtils.equals(resultStatus, "9000")) {
				// 该笔订单是否真实支付成功，需要依赖服务端的异步通知。
				Toast.makeText(PayDemoActivity.this, "支付成功", Toast.LENGTH_SHORT).show();
			} else {
				// 该笔订单真实的支付结果，需要依赖服务端的异步通知。
				Toast.makeText(PayDemoActivity.this, "支付失败", Toast.LENGTH_SHORT).show();
			}
			break;
		}
		default:
			break;
		}
	};
};
```
