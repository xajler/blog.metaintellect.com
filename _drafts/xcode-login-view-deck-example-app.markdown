## Project

* Create Empty app - dont't use Git.
* Get .gitignore and init git repository.
* Git commit.

## CocoaPods and ViewDeck
* Install CocoaPods.
* Create Podfile with ViewDeck pod in it.
* Open now workspace, add QuartzCore framework, build.
* Git commit.

## Main Storyboard

* Create a Main _Storyboard_ and save it to en.lproj folder.
* Add it to <app_name>-Info.plist with vim editor.

```xml
<key>UIMainStoryboardFile</key>
<string>Main</string>
```

* Create `LoginViewController` as a starting point of _Storyboard_, test it.
* Git commit.

## LoginViewController: Design

* Create `images` group inside `Supporting Files` group.
* Add images for Login to project, button and fields image background.
* Add fields background to LoginView.
    * Image: login_input_bg.png.
* Create two fields for Username, Password.
* Username properties:
    * Placeholder: Username.
    * Border Style: nothing (first).
    * Clear Button: Appers while editing.
    * Correction: No.
    * Return Key: Next.
    * Tag: 1.
* Password properties:
    * Placeholder: Password.
    * Border Style: nothing (first).
    * Clear Button: Appers while editing.
    * Correction: No.
    * Return Key: Go.
    * Tag: 2.
* Create Button "Log In".
* Login properties:
    * Type: Round Rect.
    * All state configs have same and background: login_button.png.
    * Disabled only has text color: pale color of desired, here gray.
* Git commit

## LoginViewController: Code

* Create `View Controllers` group in root of project.
* Create `LoginViewController` code file in `View Controllers` group.
* In Main.storyboard add
* Add to `LoginViewControllor` interface TextField Delagate (`<UITextFieldDelegate>`).
* Connect Username TextField, Password TextField, Input Fields Background and Login Button as Outlets.
    * usernameTextFileld.
    * passwwordTextField.
    * loginButton.
    * loginInputImage.
* Create IBAction on touch of Login Button named "login".
* To `login` method add:

```objective-c
if ([[self.usernameTextFileld text] isEqualToString:@"xajler"]
    && [[self.passwordTextField text] isEqualToString:@"aeon"])
{
    // TODO: Don't care for now!
}
else
{
    [self.passwordTextField setText:@""];
    [self.passwordTextField becomeFirstResponder];
}
```

* Implement `textFieldShouldReturn`:

```objective-c
- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
    if (textField.tag == 1)
    {
        UITextField *passwordTextField = (UITextField *)[self.view viewWithTag:2];
        [passwordTextField becomeFirstResponder];
    }
    else
    {
        [textField resignFirstResponder];
        // TODO: Don't care for now!
    }

    return YES;
}
```

* Implement `textFieldShouldClear`:

```objective-c
- (BOOL)textFieldShouldClear:(UITextField *)textField
{
    return YES;
}
```

* Add `touchesBegan` to hide keyboard when touched outside text field(s):

```objective-c
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    [self.view endEditing:YES];
    [super touchesBegan:touches withEvent:event];
}

```

### Enable Login button when both fields have text.

* Disable Login Button as default state: In Content uncheck Enabled.
* Add `#define` macro `allTrim`:

```objective-c
#define allTrim( object ) [object stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceCharacterSet] ]
```

* To `textFieldShouldClear` set it to be disabled.

```objective-c
- (BOOL)textFieldShouldClear:(UITextField *)textField
{
    [self.loginButton setEnabled: NO];
    return YES;
}
```
* Implement `UITextFieldDelegate` method `shouldChangeCharactersInRange`:

```objective-c
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range
                                                       replacementString:(NSString *)string
{
    if (([textField tag] == 2 && [allTrim([self.usernameTextFileld text]) length] != 0)
         || ( [textField tag] == 1 && [allTrim([self.passwordTextField text]) length] != 0))
    {
        [self.loginButton setEnabled:YES];
    }

    return YES;
}
```

## Facebook like login animation when keyboard is in focus

* Create a private instance method `_animateLoginControlsOnYAxis` (Since it is used two times):

```objective-c
- (void)_animateLoginControlsOnYAxis:(CGFloat)yInputImageValue
                   usernameTextField:(CGFloat)yUsernameValue
                   passwordTextField:(CGFloat)yPasswordValue
                         loginButton:(CGFloat)yButtonValue

{
    [UIView animateWithDuration:0.2 animations:^{
        CGRect frame;

        frame = self.loginInputImage.frame;
        frame.origin.y = yInputImageValue;
        self.loginInputImage.frame = frame;

        frame = self.usernameTextFileld.frame;
        frame.origin.y = yUsernameValue;
        self.usernameTextFileld.frame = frame;

        frame = self.passwordTextField.frame;
        frame.origin.y = yPasswordValue;
        self.passwordTextField.frame = frame;

        frame = self.loginButton.frame;
        frame.origin.y = yButtonValue;
        self.loginButton.frame = frame;
    }];
}
```
* Implement `UITextFieldDelegate` method `textFieldDidBeginEditing`. When keyboard is in focus controls go up:

```objective-c
-(void) textFieldDidBeginEditing:(UITextField *)textField
{
    [self _animateLoginControlsOnYAxis:102 usernameTextField:108 passwordTextField:147 loginButton:194];
}
```

* Change method `touchesBegan`. When keyboard is closed the controls go down or to original position.

```objectiv-c
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    [self.view endEditing:YES];
    [super touchesBegan:touches withEvent:event];
    [self _animateLoginControlsOnYAxis:162 usernameTextField:168 passwordTextField:207 loginButton:254];
}
```

* Git commit.

## Create InitialViewController

* To use a `ViewDeck` as menu in Storyboard, we need to create a `InitialViewController`.
* Create new `InitialViewController` as code into the `View Controllers` group.

### Design

* Create `InitialViewController` in _Main.storyboard_.
    * In _Identity Inspector_ set class to `InitialViewController`.
* Control + Click on `LoginViewController` _Files Owner_ icon in _Storyboard_ and drag to `InitialViewController` View.
    * In _Manual Segue_ popup choose `popup`.
    * Select on newly created _Segue_ go to _Attributes Inspector_ and for _Identifier_ enter: `LoginSegue`.

### Code

* Refactor `login` method to a new private instance method in `LoginViewController` named `_validateAndRedirect`:
    * The _Username_ ('xajler') and _Password_ ('aeon') are hard-coded and used to validate Login View.

```objective-c
- (void)_validateAndRedirect:(id)sender
{
    if ([[self.usernameTextFileld text] isEqualToString:@"xajler"]
        && [[self.passwordTextField text] isEqualToString:@"aeon"])
    {
        [self performSegueWithIdentifier:@"LoginSegue" sender:sender];
    }
    else
    {
        [self.passwordTextField setText:@""];
        [self.passwordTextField becomeFirstResponder];
    }
}

* Use `_validateAndRedirect` in refactored `login` method:

```abjective-c
- (IBAction)login:(id)sender
{
    [self _validateAndRedirect:sender];
}
```

* Remove _TODO_ from `textFieldShouldReturn` method and the call to `_validateAndRedirect` method:

```objective-c
- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
    if (textField.tag == 1)
    {
        UITextField *passwordTextField = (UITextField *)[self.view viewWithTag:2];
        [passwordTextField becomeFirstResponder];
    }
    else
    {
        [textField resignFirstResponder];
        [self _validateAndRedirect:textField];
    }

    return YES;
}
```

### ViewDeck

* In `InitialViewController` interface set the:
    * Import ViewDeck: `#import "IIViewDeckController.h"`
    * Remove inheritance to `UIViewController` and set to `IIViewDeckController`.

* In `InitialViewController` implementation add method `initWithCoder`:

```objective-c
- (id)initWithCoder:(NSCoder *)aDecoder
{
    UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
    self = [super initWithCenterViewController:[storyboard instantiateViewControllerWithIdentifier:@"MainViewController"]
                            leftViewController:[storyboard instantiateViewControllerWithIdentifier:@"LeftMenuViewController"]];
    if (self) {
        // Add any extra init code here
    }
    return self;
}
```

* We still don't have a `MainViewController` and `LeftMenuViewController` called in this code.
    * `MainViewController` is where the menu button will be triggered.
    * `LeftViewController` is the actual menu that will be shown.
* Git commit.

## Main View Controller

* Create new source file `MainViewController`.
* Create new _View Controller_ in _Storyboard_ named: `MainViewController`.
    * In _Identity Inspector_ set class to `MainViewController`.
    * Select `MainViewController` if not selected. Go to `Editor > Embed In > Navigation Controller`.
    * The _Navigation Controller Bar_ is now visible in `MainViewController`.

### Menu Bar Button Item

* Copy `ButtonMenu` image to `images` group.
* Drag new _Bar Button Item_ to the left side of `MainViewController` _Navigation Controller Bar_.
    * Remove default _Title_ (`Item`).
    * Set image to: `ButtonImage`.
* Drag _Menu Bar Item_ to `MainViewController` interface and create _IBAction_ method named: `revealLeftSidebar`.
* Change `(id)sender` to `(UIBarButtonItem *)sender`.
* Implement it:

```objective-c
- (IBAction)revealSidebar:(UIBarButtonItem *)sender {
    self.viewDeckController.leftLedge = 50.0;
    [self.viewDeckController toggleLeftViewAnimated:YES];
}
```

## Left View Controller

* Create new _View Controller_ in _Storyboard_ named: `LeftMenuViewController`.
    * In _Attributes Inspector_ set desired _Background_.



## Set StoryboardID called from Initial View Controller

* In Main _Storyboard_:
    * Select _Navigation Controllor_ of `MainViewController` set in _Identitiy Inspector_ a _Storyboard ID_ to `MainViewController`.
    * Select `LeftMenuViewController` and set in _Identity Inspector_ a _Storyboard ID_ to `LeftViewController`.

* Clean, Build and Run. Make sure the Login and View Deck menu are working.

