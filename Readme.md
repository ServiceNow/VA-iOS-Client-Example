# Example application integrating the ServiceNow Chat Client into a 3rd party app (COMING SOON!)


This is a simple app that has a button to launch the chat window. It is meant to show the minimal code necessary to integrate with the Chat Client.

For Authentication, the chat client accepts either a ServiceNow access token or an OpenID Connect (OIDC) access token. This example app uses a ServiceNow login to obtain a ServiceNow token. The token is passed into the chat service in ViewController.swift (establishChatSession).


## To add the Chat Client to your app you will have to first have [Carthage](https://github.com/Carthage/Carthage#installing-carthage) installed. Carthage is used to fetch and build the Chat Client and its dependencies

1) Create / Update your Cartfile to include SnowChat:
```   
   git "https://github.com/ServiceNow/VA-iOS-SDK"

```
   
2) Build SnowChat and dependencies: 
   From the terminal, in the directory of your Cartfile
```   
   carthage update
```   

3) Update the project by adding the build frameworks:

   * On your application targets’ General settings tab, in the “Linked Frameworks and Libraries” section, drag and drop each framework you want to use from the `Carthage/Build` folder

   * On your application targets’ Build Phases settings tab, click the + icon and choose New Run Script Phase. Create a Run Script in which you specify your shell (ex: /bin/sh), add the following contents to the script area below the shell:
```
    /usr/local/bin/carthage copy-frameworks
```
   * Add the paths to the frameworks you want to use under “Input Files”
```
    $(SRCROOT)/Carthage/Build/iOS/SnowChat.framework
    $(SRCROOT)/Carthage/Build/iOS/AMBClient.framework
    $(SRCROOT)/Carthage/Build/iOS/SlackTextViewController.framework
    $(SRCROOT)/Carthage/Build/iOS/Alamofire.framework
    $(SRCROOT)/Carthage/Build/iOS/AlamofireImage.framework
```
   * Add the paths to the copied frameworks to the “Output Files”
```
    $(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/SnowChat.framework
    $(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/AMBClient.framework
    $(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/SlackTextViewController.framework
    $(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/Alamofire.framework
    $(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/AlamofireImage.framework
```

4) In the application source code, create a ChatService instance, Establish a user session, and Present a ChatViewController
```swift
   @IBAction func startChat(_ sender: Any) {

        // create a chatService instance
        chatService = ChatService(instanceURL: instanceURL!, delegate: self)
        
        // establish the session ith the auth-token
        chatService?.establishUserSession(token: token, completion: { error in
            guard error == nil else { return }
            
            // if successful session, show the chat view controller
            self.presentChatView()
        })
    }
    
    private func presentChatView() {
        guard let chatVC = chatService?.chatViewController() else { return }
        
        presentInNavigationController(chatVC)
    }
    
    // optional
    private func presentInNavigationController(_ chatVC: ChatViewController) {
        let localizedDoneString = NSLocalizedString("Done", comment: "Done button")
        let doneButton = UIBarButtonItem(title: localizedDoneString, style: .done, target: self, action: #selector(finishChatPresentation(_:)))
        chatVC.navigationItem.leftBarButtonItem = doneButton
        
        let navigationController = UINavigationController(rootViewController: chatVC)
        navigationController.modalPresentationStyle = .overFullScreen
        
        present(navigationController, animated: true, completion: nil)
    }
    
    // optional
    @objc func finishChatPresentation(_ sender: AnyObject) {
        chatVC?.dismiss(animated: true, completion: nil)
    }
    
    // required
    func chatServiceAuthenticationDidBecomeInvalid(_ chatService: ChatService) {
        finishChatPresentation(self)
    }
```    

