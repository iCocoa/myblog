---
title: Difference between authentication, authorization,verification, validation
date: 2017-12-12 21:41:34
tags:
---
`verification`, `validation`, `authentication`, `authorization` 这几个术语很常用，也经常被误用，这里做一次对比总结。

**identity**
>A security principal (you or a computer, typically) wants to access a system. Because the system doesn’t know you yet, you need to make a declaration of who you are. Your answer to the question “Who are you” is the first thing you present to a system when you want to use it. Some common examples of identity are user IDs, digital certificates (which include public keys), and ATM cards. A notable characteristic of identity is that it is public, and it has to be this way: identity is your claim about yourself, and you make that claim using something that’s publicly available.

*身份* 指的是具有公共属性的身份，表示安全主体是谁。

**authentication**

>This is the answer to the question “OK, how can you prove it?” When you present your identity to a system, the system wants you to prove that it is indeed you and not someone else. The system will challenge you, and you must respond in some way. Common authenticators include passwords, private keys, and PINs. Whereas identity is public, authentication is private: it’s a secret known (presumably) only by you. In some cases, like passwords, the system also knows the secret. In other cases, like PKI, the system doesn’t need to possess the secret, but can validate its authenticity (this is one of many reasons why PKI is superior). Your possession of this secret is what proves that you are who you claim to be.

*认证* 指的是如何证明你就是某人，强调的是证明的过程、手段、机密性证据。

**authorization**
>Once you’ve successfully authenticated yourself to a system, the system controls which resources you’re allowed to access. Typically this is through the use of a token or ticket mechanism. The token or ticket constrains your ability to roam freely throughout the system. By “caching” your authenticated identity for subsequent access control decisions, it allows you to access only that which the administrators have determined is necessary, thus enforcing the principle of least privilege.

*授权* 指的是授予某主体执行某些操作或获取某些资源的权利。

**validation**
>To check data or filter data that requires no external references; usually meaning to check the format of the data matching a particular pattern. For example, check if something is filled in or not or the pattern of an email address matches. More specifically, validation is doing as little work as possible to check the very basic assumptions of the data.

*验证* 确保数据的真实性、有效性，在理论及逻辑层面上进行。例如：确保电话号码是否有效真实。

**verification**
>Occurs after Validation in that it is more complex and you would always use validation first and not allow verification if the validation did not pass. Verification has to do with checking against a current set of data that takes more resources to discover than Validation. For example, checking if an email has already been registered or not requires a lookup of existing registered users or another example would be checking if a zip code entered is actually a real zipcode by looking through a database of registered zipcodes.

*校验、核对、核实* 往往是两个以上个体之间进行对比。例如：手机验证码用来确保是机主本人。

参考：

[It’s Me, and Here’s My Proof: Why Identity and Authentication Must Remain Distinct](https://technet.microsoft.com/en-us/library/cc512578.aspx)

[Difference between “validation” and “verification”](https://english.stackexchange.com/questions/53866/difference-between-validation-and-verification)




