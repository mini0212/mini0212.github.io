---
title: "mailto와 MFMailComposeViewController"
category: "Swift"
---

회사에서 cs문의 이메일 폼을 만들다가 궁금해서 찾아봄

mailto - **메일 앱을 통해** 메일을 보내는 것

MFMailComposeViewController - **앱 내에서** 메일을 보내는 것

메일 계정(설정 - 암호 및 계정)이 없을 경우 MFMailComposeViewController에서 false를 반환

```swift
class mailTest: MFMailComposeViewControllerDelegate {
    func sendMail() {
        let email = "ksmini0212@gmail.com"
        let subject = "Hello."
        let bodyText = "It's me"     
    
        if MFMailComposeViewController.canSendMail() {
            // 메일 계정이 있을 경우
            let composeVC = MFMailComposeViewController()
            composeVC.delegate = self as? UINavigationControllerDelegate
            composeVC.setToRecipients([email])
            composeVC.setMessageBody(bodyText, isHTML: false)
            self.present(composeVC, animated: true, completion: nil)
        } else {
            let coded = "mailto:\(email)?subject=\(subject)&body=\(bodyText)"
                        .addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)
            if let url = URL(string: coded!) {
                UIApplication.shared.open(url, options: [:], completionHandler: nil)
            }
        }
    }
}
```

계정이 없다면 메일 앱으로 이동하여 메일을 등록하라는 화면이 뜬다.

MFMail을 사용하면 메일을 보내거나 취소해도 dismiss가 안된다..(머쓱)

+) 메일작성화면에서 첨부파일 추가하는 방법
iOS12는 오버팝 후 > 누르면 첨부 버튼이 나옴
iOS13은 하단 액세서리 뷰에서 파일 첨부 가능


