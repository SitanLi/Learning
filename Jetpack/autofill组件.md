### AutoFill组件

包含了一系列自动填充服务所需要的组件，`HintConstants`，通过组合这些常量，可以构建自动填充场景。

自动填充服务Api地址：https://developer.android.com/guide/topics/text/autofill-optimize

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.autofill:autofill:1.0.0"
}
```

#### HintConstants 常量列表

| 字段                                              | 描述                                   |
| :------------------------------------------------ | :------------------------------------- |
| AUTOFILL_HINT_BIRTH_DATE_DAY                      | 指示此试图可以自动填充出生日期（日期） |
| AUTOFILL_HINT_BIRTH_DATE_FULL                     | 出生日期（完整的）                     |
| AUTOFILL_HINT_BIRTH_DATE_MONTH                    | 出生日期（月份）                       |
| AUTOFILL_HINT_BIRTH_DATE_YEAR                     | 出生日期（年份）                       |
| AUTOFILL_HINT_CREDIT_CARD_EXPIRATION_DATE         | 信用卡有效期                           |
| AUTOFILL_HINT_CREDIT_CARD_EXPIRATION_DAY          | 信用卡有效期（日期）                   |
| AUTOFILL_HINT_CREDIT_CARD_EXPIRATION_MONTH        | 信用卡有效期（月份）                   |
| AUTOFILL_HINT_CREDIT_CARD_EXPIRATION_YEAR         | 信用卡有效期（年份）                   |
| AUTOFILL_HINT_CREDIT_CARD_NUMBER                  | 信用卡卡号                             |
| AUTOFILL_HINT_CREDIT_CARD_SECURITY_CODE           | 信用卡安全码                           |
| AUTOFILL_HINT_EMAIL_ADDRESS                       | 电子邮件地址                           |
| AUTOFILL_HINT_GENDER                              | 性别                                   |
| AUTOFILL_HINT_NEW_PASSWORD                        | 保存\|更新密码                         |
| AUTOFILL_HINT_PASSWORD                            | 自动输入密码                           |
| AUTOFILL_HINT_PERSON_NAME                         | 自动填充全名                           |
| AUTOFILL_HINT_PERSON_NAME_FAMILY                  | 自动填充姓氏                           |
| AUTOFILL_HINT_PERSON_NAME_GIVEN                   | 自动填充名字                           |
| AUTOFILL_HINT_PERSON_NAME_MIDDLE                  | 自动填充中间名                         |
| AUTOFILL_HINT_PERSON_NAME_MIDDLE_INITIAL          | 自动填充中间名的缩写                   |
| AUTOFILL_HINT_PERSON_NAME_PREFIX                  | 自动填充名字前缀                       |
| AUTOFILL_HINT_PERSON_NAME_SUFFIX                  | 自动填充名字后缀                       |
| AUTOFILL_HINT_PHONE_COUNTRY_CODE                  | 自动填充电话号码国家/地区代码          |
| AUTOFILL_HINT_PHONE_NATIONAL                      | 自动填充不带国家/地区的电话号码        |
| AUTOFILL_HINT_PHONE_NUMBER                        | 自动填充完整电话号码以及国家/地区代码  |
| AUTOFILL_HINT_PHONE_NUMBER_DEVICE                 | 自动填充当前设备的电话号码             |
| AUTOFILL_HINT_POSTAL_ADDRESS                      | 自动填充邮件地址                       |
| AUTOFILL_HINT_POSTAL_ADDRESS_COUNTRY              | 自动填充国家名称/代码                  |
| AUTOFILL_HINT_POSTAL_ADDRESS_EXTENDED_ADDRESS     | 自动填充地址详细信息（扩展）           |
| AUTOFILL_HINT_POSTAL_ADDRESS_EXTENDED_POSTAL_CODE | 自动填充邮政编码（扩展）               |
| AUTOFILL_HINT_POSTAL_ADDRESS_LOCALITY             | 自动填充地址位置（城市）               |
| AUTOFILL_HINT_POSTAL_ADDRESS_REGION               | 自动填充地址区域                       |
| AUTOFILL_HINT_POSTAL_ADDRESS_STREET_ADDRESS       | 自动填充地址街道地址                   |
| AUTOFILL_HINT_POSTAL_CODE                         | 自动填充邮政编码                       |
| AUTOFILL_HINT_SMS_OTP                             | 自动填充短信一次性密码                 |
| AUTOFILL_HINT_USERNAME                            | 自动填充用户名                         |

`generateSmsOtpHintForCharacterPosition()`：该方法指示此试图可以用SMS一次性密码的第几个字符/数字自动填充。

#### 版本更新

##### 版本1.1.0

- 添加一组 API 以支持构建自动填充内嵌建议（Android 11 中引入的一项新功能）。如需了解详情，请参阅 [IME 自动填充指南](https://developer.android.com/guide/topics/text/ime-autofill)。
- 推出了 v1 界面模板 [inlineSuggestionUi](https://developer.android.com/reference/androidx/autofill/inline/v1/InlineSuggestionUi)，以帮助 IME 开发者指定内嵌建议样式，并帮助自动填充提供程序构建内嵌建议内容。

##### 版本1.0.0

- 自动填充模块的首个稳定版本。
- 添加一组标准的受支持自动填充提示常量，这些常量应被所有自动填充服务支持。