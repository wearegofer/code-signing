# macOS Code Signing

To use macOS code signing we must hold a valid apple developer id.
Currently, **Ran Magen** is the owner of the apple id and manages the team.
Ran can invite anyone else to be a team member so he can create a new signing certificate.

Currently, our subscription is untill ***13/12/2019*** and must be renewed before hand.

# What is this certificate?
When joining to the `Apple Developer Program` we get access to some resources, the two ones that we use are:
- Publish Profile
- Certificate


Those resources allow us to sign and publish our code and to be shown to the users as a legit program, `Apple's macOS` has a similar software mechanisem to the [`Microsoft Windows Defender SmartScreen`](https://github.com/wearegofer/code-signing/blob/master/Windows.md#what-is-this-certificate).

When downloading an application from the internet or a source which is not the `Apple AppStore` we will recive a message indicated that the program is not yet fully trusted.

When the application is both unsigned and was downloaded from an untrusted source the following screen will appear
![Unsigned and untrusted](https://www.technipages.com/wp-content/uploads/2017/01/MacOS-Gateway-Warning-1280x720.png)

If the application was signed, but was still downloaded from an untrusted source, the following popup will occur.
![Signed and untrusted](https://support.apple.com/library/content/dam/edam/applecare/images/en_US/macos/Mojave/macos-mojave-notarized-app-alert-dark.jpg)

Same as `SmartScreen` when the app will get enough downloads, `macOS` will mark it as safe, and therefore will not display this warning.

## How long the certificate is valid to?

`Apple Developer Program` is on an early subscription, which means it should be renewed each year, and also re-generate the certifiacte as well.
Currently we need to renew the program at the ***13/12/2019***

## Renewing the Certificate
To renew the certificate we must have a valid `Apple Developer Subscription` and beign a member of the team.

Then we need to go to the [Apple Developer Dashboard](https://developer.apple.com/),
We will select `Certificates, IDs & Profiles`, 

## Generating the `.p12` Certificate
This step requires us to have certain pre-requisits:
- `macOS` based PC with our `Developer Apple ID` connected
- XCode must be installed
- A valid `Apple Developer Program` subscription

We will open the `XCode` app in the `macOS` computer, then we will go to `perferences` (`Alt+,` or `Option (⌥) + 
,`)

![XCode Perferences](https://i.gyazo.com/89258d5a2f23489de1c91dbdb532366f.png)

Then we will go to the `Accounts` tab, select our team - `GOFER TECHNOLOGIES LTD` and click `Manage Certificates`, we will be greeted by the `Signing certificates for "GOFET TECHNOLOGIES LTD"` window.

![Signing Certificates window](https://i.gyazo.com/4e7a54824aa5cb3e79cf01e63e0aac29.png)

We will need to select the following **3** valid certificates:
- `Mac Installer Distribution`
- `Developer ID Application`
- `Developer ID Installer`

Then we will multi select them and `right click -> export`.
We will need to select a destination and name for the `.p12` file, we will click `Save`

![Save Dialog](https://i.gyazo.com/70642cf90832f0ec4bd724b36914e992.png)

We will be forced to select a `passphrase` to protect the exported certifaces, those certificates also include the `private key`, so in a case they get stolen, we don't want people just beign able to sign code under our identity.

![Passphrase dialog](https://i.gyazo.com/016ee844b2b7af7ecbfdfed11ccca58a.png)

The current `passphrase` can be found under the `variables` tab for each `macOS build job` in our [Azure DevOps](https://wearegofer.visualstudio.com/Gofer/_apps/hub/ms.vss-ciworkflow.build-ci-hub?_a=edit-build-definition&id=19&view=Tab_Variables) under the `CSC_KEY_PASSWORD` variable.


## Uploading the certificate to the build agents

For the build agent to be able to proccess the certificate it must be able to do the following:
- Downloading the `.p12` file
- Opening the `.p12` file and exporitng its private key using the passphrase.

To store the `.p12` file securly and it to available to the different build agents we use the `Secure Files` options in the Microsoft Azure DevOps that can be found under the following URI:
[https://wearegofer.visualstudio.com/Gofer/_library?itemType=SecureFiles](https://wearegofer.visualstudio.com/Gofer/_library?itemType=SecureFiles)
We can upload new certifiactes as we would like, **Note** that you can't replace an existing file, so we first must need to chagne the current file file name, afterwards uploading the new one.

## Updating the Build Jobs

Each job requires 3 **steps** to sign the code:
- Download the secure file
- Getting the vairable to know where the secure file is hosted
- Unpacking the certificate using the provided passphrase

We would need to go each job and upadte it manually to match the code signing modifications.
We will first need to update the `Download Secure File` step and select the new certificate file from the `Secured Files` valut.

Afterwards, we will need to update 2 variables under the `Pipeline variables` section
- CSC_KEY_PASSWORD
- CSC_LINK

### CSC_LINK
This variable will contain the path to the certificate after it beign downloaded by the `Download Secure File` step,
It would usally look something like this:
```
$(Agent.WorkFolder)/_temp/<CertFileName>.p12
```
while the `<CertFileName>` will be the name of the fileiself without the .pfx extension

### CSC_KEY_PASSWORD
This variable will contain the `passphrase` that we set earlier while exporting the certificate as `.p12` file.
It would allow the job to unpack the certificate and its private key to sign on top of the code.

Once we modified all the jobs, we can now build the code and it should be signed.
