# Windows Code Signing

We use [DigiCert](https://www.digicert.com) to manage our windows code signing certificates.
The following users has access to manage the account and re-issue or buy new certificats.
- Hagai Bloch (mrx@wearegofer.com)
- Chen Machluf (machluf@wearegofer.com)
- Ran Magen (ran@wearegofer.com)
- DevOps Account (devops@wearegofer.com)

# What is this certificate?

Windows code signing certificate allows us to generate a digital signiture on the `Gofer` app when releasing it to the public.
Microsoft allows certain resellers, such as `DigiCert` to issue those certificates, when executing an `Zone 3` file, which basicly says its from the WWW and not issued by the current computer domain, `Microsoft SmartScreen` pops up.
This service is responsible to validate each application and decided if to run it automatilcy, prompt the user for confirmation nor block it completly.
`Zone 3` apps which aren't signed by a digital code signing certification will be shown as `Unknown publisher` under the `SmartScreen` window.

![Smart Screen Unknown publisher](https://user-images.githubusercontent.com/6978458/34665337-de082852-f470-11e7-81d0-50ed35a4278c.png)

In some cases `SmartScreen` will also classify the app as ***dangerous*** and will show a red screen while hiding the `Run Anyway` button.

![Blocked app from smart screen](https://user-images.githubusercontent.com/11667494/29426086-81b8ff24-8353-11e7-9300-e7db1c0c5dc6.png)

If the certificate is valid, a normal prompt with the app publisher details will be displayed.

![Smart screen with publisher info](https://i.stack.imgur.com/YRQme.jpg)

Why we still get the same screen?
If the code signing isn't a `Extended Validation`, EV for short, which costs more and requires a physical 2fa usb drive to be present while signing in the build server, Microsoft will require some time to validate the information and afterward it will be shown with out smart screen.

## How long the certificate is valid to?

each certificate can be bought in advance to a maximum period of **3 years**.
Our certificate as for the **09/09/2019** is valid thorugh the **15/01/2020**.
But we must renew it before hand, at least two weeks in advance, enought time to rollout the new certificate.

On the next renewal we would need to decide the next term of the code-signing certificate validation.

## Renewing the Certificate

Once the time have come, and we would need to renew the certificate, we will log-in to the **DigiCert** control panel and would renew the certificate.
We must see that we choose the **Code Signing** certificate type and choosing it to be `Microsoft Authenticode`.
Once the order will be proccessed, the account that issued the request, will recive a link to create the certificate file.
We will recive the `.cert` file after the process is finished,
At this time we will be required to export it as `.pfx` file so the build server could be using, the `.pfx` file also uses a passphrase to not allow any other user to get the `.cert` and use it by himself.

## CERT --> PFX
It would be best to follow the **DigiCert** offical guide and tutorial that can be found [here](https://www.digicert.com/util/pfx-certificate-management-utility-import-export-instructions.htm).

> It is highly important that we will export the .pfx file **including  the private key**

The password is a very important part, that needs to be remembered and also to be provided under the build config in the build agents.
I wouldn't provide the passphrase here, but it can be found under the `CSC_KEY_PASSWORD` variable in anyone of the `Windows Build` agents (Production and Staging alike)

## Uploading the certificate to the build agents

For the build agent to be able to proccess the certificate it must be able to do the following:
- Downloading the `.pfx` file
- Opening the `.pfx` file and exporitng its private key using the passphrase.

To store the `.pfx` file securly and it to available to the different build agents we use the `Secure Files` options in the Microsoft Azure DevOps that can be found under the following URI:
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
$(Agent.WorkFolder)/_temp/<CertFileName>.pfx
```
while the `<CertFileName>` will be the name of the fileiself without the .pfx extension

### CSC_KEY_PASSWORD
This variable will contain the `passphrase` that we set earlier while exporting the certificate as `.pfx` file.
It would allow the job to unpack the certificate and its private key to sign on top of the code.

Once we modified all the jobs, we can now build the code and it should be signed.
