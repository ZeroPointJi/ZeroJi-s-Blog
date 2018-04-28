---
title:  "iOS_ApplyPay"
date:   2016-06-24 10:00:00 +8:00
categories: 
  - iOS
tags: 
  - Objective-C
---
### ApplyPay

`ApplyPay` 是苹果推出的支付方式, 意在创建更加快捷方便的支付.

`ApplyPay` 需要后台前台相互协作, 而在文章中, 我们只涉及怎么做好前端.

### ApplyPay 环境配置

`ApplyPay` 的使用需要开发者证书和商用ID, 这些需要在苹果开发者设置中创建.

#### 开发者证书的获取

首先打开应用程序 `KeyChain(钥匙串访问)`, 在左上角钥匙串访问->证书助理中, 选择 `从证书颁发机构求证书`.

在 `用户电子邮件地址` 一栏填写你的开发者账号, `常用名称` 随便填写, `CA电子邮件地址` 一栏不需要填写. 然后选择 `储存到磁盘`.

选择路径储存证书请求文件.

这个文件在下面会用到.

#### 商用ID申请

登录 [https://developer.apple.com/](https://developer.apple.com/), 选择 `Account` 选项, 填写开发者账号密码进入设置界面.

1. 在设置页面, 选择 `Certificates，Identifiers&Profiles`
2. 在 `Identifiers` 下, 选择 Merchant IDs`
3. 在右上角点击 `+` 按钮
4. 在 `Description` 栏 输入相应信息, 点击 `Continue`
5. 浏览下配置参数, 点击 `Register'
6. 点击 'Done` 完成

为你的ID标明配置一个证书

1. 在设置页面, 选择 `Certificates，Identifiers&Profiles`
2. 在 `Identifiers` 下, 选择 Merchant IDs`
3. 选择刚才创建的 `ID`, 点击 `Edit`
4. 点击 `Create certificate`, 点击 `Conitnue`
5. 点击 `Choose File`, 选择之前创建的证书请求, 点击 `Generate`
6. 点击 `Download` 下载证书, 点击 `Done` 完成

这样, 证书生成完毕.

### 配置App使其支持ApplyPay

为了在 `Xcode` 中启用 `Apply Pay`, 打开 App 工程文件的 `Capabilities` 面板. 在 `Apple Pay` 这行将开关按钮设置为 `ON`, 接着选择 App 需要使用的 ID 标示.

### ApplyPay代码

在 `ViewController` 中引入两个类

1. `PassKit/PassKit.h` 用户绑定银行卡的信息
2. `PassKit/PKPaymentAuthorizationViewController.h` `Apply Pay` 的展示控件

创建一个属性用于储存模拟卡片, 并且懒加载模拟卡片

```objc
/** 支持的卡片类型 */
@property (nonatomic, strong) NSArray *supportNewworks;

 (NSArray *)supportNewworks
{
    if (!_supportNewworks) {
        // 此处只是模拟卡片, 真实卡片需要获取
        self.supportNewworks = [NSArray arrayWithObjects:PKPaymentNetworkAmex, PKPaymentNetworkMasterCard, PKPaymentNetworkVisa, PKPaymentNetworkChinaUnionPay, nil];
    }
    
    return _supportNewworks;
}
```

在使用 `Apply Pay` 之前, 我们需要判断设备是否支持 `Apply Pay`, 是否有绑定支持 `Apply Pay` 的卡片

```objc
// 是否能使用 Apply Pay
- (BOOL)canMakePay
{
    // 判断当前设备是否支持 Apply Pay
    if (![PKPaymentAuthorizationViewController canMakePayments]) {
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"警告" message:@"当前设备部支持 Apply Pay" preferredStyle:UIAlertControllerStyleActionSheet];
        UIAlertAction *action = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
            [alert dismissViewControllerAnimated:YES completion:nil];
        }];
        [alert addAction:action];
        [self presentViewController:alert animated:YES completion:nil];
        
        return NO;
    }
    
    #warning 当真机调试时 如果这里canMakePaymentsUsingNetworks不能通过，表示手机的钱包（wallet）还没有开通，开通后，添加一张卡，就可以使用
    // 判断是否有绑定支持的卡片
    if (![PKPaymentAuthorizationViewController canMakePaymentsUsingNetworks: self.supportNewworks]) {
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"警告" message:@"没有绑定任何支持类型的卡片, 请绑定" preferredStyle:UIAlertControllerStyleActionSheet];
        UIAlertAction *action = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
            [alert dismissViewControllerAnimated:YES completion:nil];
        }];
        [alert addAction:action];
        [self presentViewController:alert animated:YES completion:nil];
        
        return NO;
    }
    
    return YES;
}
```

配置支付请求方法 

```objc
// 配置支付请求
- (void)setupRequest
{
    // 创建请求
    PKPaymentRequest *request = [[PKPaymentRequest alloc] init];
    // 创建商品
    PKPaymentSummaryItem *item = [PKPaymentSummaryItem summaryItemWithLabel:@"诺心食品有限公司" amount:[NSDecimalNumber decimalNumberWithString:@"1"]];
    
    // 设置请求
    request.paymentSummaryItems = @[item]; // 商品信息
    request.countryCode = @"CN"; // 国家
    request.currencyCode = @"CNY"; // 金额显示格式
    request.supportedNetworks = _supportNewworks; // 支持的支付方式
    request.merchantIdentifier = @"merchant.lecakeApplePay";
    request.merchantCapabilities = PKMerchantCapability3DS|PKMerchantCapabilityEMV; // 支持的交易处理协议
    
    // 创建支付界面
    PKPaymentAuthorizationViewController *paymentSheet = [[PKPaymentAuthorizationViewController alloc] initWithPaymentRequest:request];
    if (paymentSheet) {
        [self presentViewController:paymentSheet animated:YES completion:nil];
        // 设置代理
        paymentSheet.delegate = self;
    }
}
```

接受 `PKPaymentAuthorizationViewControllerDelegate` 代理, 实现代理方法.

```objc
// 支付成功后, 苹果服务器返回信息回调
- (void)paymentAuthorizationViewController:(PKPaymentAuthorizationViewController *)controller didAuthorizePayment:(PKPayment *)payment completion:(void (^)(PKPaymentAuthorizationStatus))completion
{
    // 支付凭证, 发给服务器验证支付是否真实有效
    //PKPaymentToken *paymentToken = payment.token;
    //PKContact *billingContact = payment.billingContact; //账单信息
    //PKContact *shippingContact = payment.shippingContact; //送货信息
    //PKShippingMethod *shippingMethod = payment.shippingMethod; //商品信息
    
    // 等待服务器返回结果后, 再进行系统 block 调用
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        // 模拟服务器通信
        completion(PKPaymentAuthorizationStatusSuccess);
    });
}

// 支付完成回调
- (void)paymentAuthorizationViewControllerDidFinish:(PKPaymentAuthorizationViewController *)controller
{
    [controller dismissViewControllerAnimated:YES completion:nil];
}
```

最后添加一个按钮响应支付

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    if ([self canMakePay]) {
        CGRect scrrenFrame = [UIScreen mainScreen].bounds;
        CGFloat scrrenW = scrrenFrame.size.width;
        
        UIButton *payButton = [UIButton buttonWithType:UIButtonTypeCustom];
        payButton.frame = CGRectMake(100, 100, scrrenW - 200, 50);
        [payButton setTitle:@"Apply Pay" forState:UIControlStateNormal];
        [payButton setTitleColor:[UIColor darkGrayColor] forState:UIControlStateNormal];
        [payButton setBackgroundColor:[UIColor blackColor]];
        [payButton addTarget:self action:@selector(setupRequest) forControlEvents:UIControlEventTouchUpInside];
        [self.view addSubview:payButton];
    }
    
}
```

--- 

至此, 一个简单的 `ApplyPayDemo` 就此完成, github上有我的代码, 地址为[https://github.com/qq1216137037/ApplyPayDemo](https://github.com/qq1216137037/ApplyPayDemo).

参考资料: 

1. [Apple Pay编程指南](http://www.open-open.com/lib/view/open1422324034345.html)
2. [iOS App集成Apple Pay 编程指南（中国版）](http://www.jianshu.com/p/9ec40755ba35)
3. [IOS-APP提交上架流程](http://www.th7.cn/Program/IOS/201603/771495.shtml)
