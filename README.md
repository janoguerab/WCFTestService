# How to: Use Certificate Authentication and Message Security in WCF Calling from Windows Forms

## Applies to
Microsoft Windows Communication Foundation (WCF) 3.5
Microsoft Visual Studio 2008

## Summary
This how-to article walks you through the process of using client certificates and message security to authenticate your users. The article shows you how to create and install client and service certificates during development, configure the WCF service and client to use the respective certificates, and test the service with a sample WCF client.

# Contents
## Objectives

# Overview
## Summary of Steps
Step 1: Create a Sample WCF Service
Step 2: Configure wsHttpBinding with Certificate Authentication and Message Security
Step 3: Create and Install a Service Certificate
Step 4: Configure the Service Certificate for the WCF Service
Step 5: Create a Test Client
Step 6: Add a WCF Service Reference to the Client
Step 7: Create and Install the Client Certificate for Authentication
Step 8: Configure the Client Certificate in the WCF Client Application
Step 9: Test the Client and WCF Service


# Additional Resources

## Objectives

Learn how to create and use a temporary certificate for authentication and message security.
Learn where to store the temporary certificate.
Learn how to troubleshoot common errors related to temporary certificates, authentication, and message security in WCF.
Overview
When developing a WCF service that uses X.509 certificates to provide client authentication and message security, it is necessary to work with temporary certificates. This is because production certificates are expensive and may not be readily available. There are two options for specifying trust on a certificate:
Peer trust validates the certificate directly.
Chain trust validates the certificate against the issuer of a certificate known as a root authority.
This how-to article discusses the chain trust option because it is the most commonly used approach in Business-to-Business (B2B) scenarios.
To use chain trust validation during development time, you create a self-signed root certificate authority (CA) and place it in the Trusted Root Certification Authority store of the client and service machines. The certificate used by the WCF client for client authentication and the WCF service for service authentication and message protection is then created and signed by the root self-signed certificate and installed in the LocalMachine store.
You will use makecert.exe to create a certificate to act as your root CA. You will then use your root CA certificate to sign additional certificates for your WCF service and client. Finally, you will configure the WCF client and service to use your temporary certificate.
Summary of Steps
Step 1: Create a Sample WCF Service
Step 2: Configure wsHttpBinding with Certificate Authentication and Message Security
Step 3: Create and Install a Service Certificate
Step 4: Configure the Service Certificate for the WCF Service
Step 5: Create a Test Client
Step 6: Add a WCF Service Reference to the Client
Step 7: Create and Install the Client Certificate for Authentication
Step 8: Configure the Client Certificate in the WCF Client Application
Step 9: Test the Client and WCF Service
Step 1: Create a Sample WCF Service
In this step, you create a WCF service in Visual Studio.
In Visual Studio, on the File menu, click New Web Site.
In the Templates section, select WCF Service. Make sure that the Location is set to Http and specify the virtual directory to be created in the Path (e.g., http://localhost/WCFTestService).
In the New Web Site dialog box, click OK to create a virtual directory and a sample WCF service.
Browse to your WCF service (i.e., http://localhost/WCFTestService/Service.svc).
You should see details of your WCF service.
Step 2: Configure wsHttpBinding with Certificate Authentication and Message Security
In this step, you configure the WCF service to use certificate authentication and message security.
Right-click the Web.config file of the WCF service and then choose the Edit WCF Configuration option.
In the Configuration Editor, in the Configuration section, expand Service and then expand Endpoints.
Select the first node [Empty Name] and set the Name attribute to wsHttpEndpoint.
By default, the name will be empty because it is an optional attribute.
Click the Identity tab and then delete the Dns attribute value.
In the Configuration Editor, select the Bindings folder.
In the Bindings section, choose New Binding Configuration.
In the Create a New Binding dialog box, select wsHttpBinding.
Click OK.
Set the Name of the binding configuration to some logical and recognizable name; for example, wsHttpEndpointBinding.
Click the Security tab.
Make sure that the Mode attribute is set to Message, which is the default setting.
Set the MessageClientCredentialType to Certificate by selecting this option from the drop-down list.
In the Configuration section, select the wsHttpEndpoint node.
Set the BindingConfiguration attribute to wsHttpEndpointBinding by selecting this option from the drop-down list.
This associates the binding configuration setting with the binding.
In the Configuration Editor, on the File menu, select Save.
In Visual Studio, open your configuration and comment out the identity element. It should look as follows:


Copy
      <!--<identity>
        <dns value="" />
      </identity>-->
In Visual Studio, verify your configuration. The configuration should look as follows:


Copy
…
<bindings>
  <wsHttpBinding>
    <binding name="wsHttpEndpointBinding">
      <security>
        <message clientCredentialType="Certificate" />
      </security>
    </binding>
  </wsHttpBinding>
</bindings>
<services>
  <service behaviorConfiguration="ServiceBehavior" name="Service">
    <endpoint address="" binding="wsHttpBinding"
      bindingConfiguration="wsHttpEndpointBinding"
      name="wsHttpEndpoint" contract="IService">
      <!--<identity>
        <dns value="" />
      </identity>-->
    </endpoint>
    <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange" />
  </service>
</services>
…
Step 3: Create and Install a Service Certificate
In this step, you create a temporary service certificate and install it in the local store. This certificate will be used for service authentication and to encrypt the message, thereby protecting any other sensitive data.
Creating and installing the certificate is outside the scope of this How To article. For detailed steps on how to do this, see “How To - Create and Install Temporary Certificates in WCF for Message Security During Development.”
 Note
If you are running on Microsoft Windows XP, give the certificate permissions for the ASPNET identity instead of the NT Authority\Network Service identity because the Internet Information Services (IIS) process runs under the ASPNET account.
The temporary certificate should be used for development and testing purposes only. For actual production deployment, you will need to obtain a valid certificate from a certificate authority (CA).
Step 4: Configure the Service Certificate for the WCF Service
In this step, you configure the WCF service to use the temporary certificate you created in the previous step.
In the Configuration Editor, expand the Advanced node, and then expand the Service Behaviors node.
Click Add.
In the Service Behavior Element Extensions dialog box, select the serviceCredentials option and then click Add.
Expand the serviceCredentials node and then select the serviceCertificate node.
Set the FindValue attribute to the name of the service certificate that you have created; for example, "CN=tempCertServer".
Leave the default settings for StoreLocation and StoreName.
In the Configuration Editor, on the File menu, click Save.
In Visual Studio, verify your configuration. The configuration should look as follows.


Copy
...
<behaviors>
  <serviceBehaviors>
    <behavior name="ServiceBehavior">
      <serviceMetadata httpGetEnabled="true" />
      <serviceDebug includeExceptionDetailInFaults="false" />
      <serviceCredentials>
        <serviceCertificate findValue="CN=tempCertServer" />
      </serviceCredentials>
    </behavior>
  </serviceBehaviors>
</behaviors>
...
Step 5: Create a Test Client
In this step, you create a Windows Forms application to test the WCF service.
Right-click your solution, click Add, and then click New Project.
In the Add New Project dialog box, in the Templates section, select Windows Forms Application.
In the Name field, type Test Client and then click OK.
Step 6: Add a WCF Service Reference to the Client
In this step, you add a reference to your WCF service.
Right-click your Client project and then click Add Service Reference.
In the Add Service Reference dialog box, set the URL to your WCF Service (e.g., http://localhost/WCFTestService/Service.svc) and then click Go.
In the Web reference name field, change ServiceReference1 to WCFTestService.
Click Add Reference.
A reference to WCFTestService should appear beneath Web References in your Client project.
Step 7: Create and Install the Client Certificate for Authentication
In this step, you create a temporary client certificate by using the root CA created in Step 3 above, and install it in the local store. This certificate will be used for client authentication and to encrypt the message, thereby protecting any other sensitive data.
Copy the root CA certificate (RootCATest.cer) and private key file (RootCATest.pvk), created as part of Step 3, to the client machine.
Open a Visual Studio command prompt and browse to the location where you copied the root CA certificate and private key file.
Run the following command for creating a certificate signed by the root CA certificate:


Copy
makecert -sk MyKeyName -iv RootCATest.pvk -n "CN=tempCert" -ic RootCATest.cer -sr CurrentUser -ss my -sky signature -pe tempCert.cer
In the Enter Private Key Password dialog box, enter the password for the root CA private key file created as part of the Step 3 above, and then click OK.
For more information and detailed steps, see “How to: Create and Install Temporary Certificates in WCF for Message Security During Development.”
Step 8: Configure the Client Certificate in the WCF Client Application
In this step, you configure the WCF client to use the temporary certificate you created in the previous step.
In your test client, right-click the App.config file and then click Edit WCF Configuration.
In the Configuration Editor, expand the Advanced node, select Endpoint Behaviors, and then select New Endpoint Behavior Configuration.
Click Add.
In the Adding Behavior Element Extension Sections dialog box, select clientCredentials and then click Add.
Expand the clientCredentials node, select the clientCertificate node, and then set the FindValue attribute to the subject name of the client certificate that you created and installed in Step 7; for example, "CN=tempCertClient".
Leave the default StoreLocation attribute set to CurrentUser as is.
In the Configuration Editor, expand the Client node, expand the Endpoints node, and then select the wsHttpEndpoint node.
Set the BehaviorConfiguration attribute to NewBehavior by choosing this option from the drop-down list.
This is the endpoint behavior you just created.
In the Configuration Editor, on the File menu, click Save.
In Visual Studio, verify your configuration. The configuration should look as follows.


Copy
<system.serviceModel>
    <behaviors>
        <endpointBehaviors>
            <behavior name="NewBehavior">
                <clientCredentials>
                    <clientCertificate findValue="CN=tempCertClient"/>
                </clientCredentials>
            </behavior>
        </endpointBehaviors>
    </behaviors>
    ...
    <client>
        <endpoint address="http://<<service address>>"
            behaviorConfiguration="NewBehavior" binding="wsHttpBinding"
            bindingConfiguration="wsHttpEnpoint1" contract="ServiceReference1.IService"
            name="wsHttpEndpoint">
            <identity>
                <certificate encodedValue="<<Encode Value>>" />
            </identity>
        </endpoint>
    </client>
</system.serviceModel>
Step 9: Test the Client and WCF Service
In this step, you access the WCF service, pass the user credentials, and make sure that the username authentication works.
In your Client project, drag a button control onto your form.
Double-click the button control to show the underlying code.
Create an instance of the proxy and call the GetData operation of your WCF service. The code should look as follows:


Copy
private void button1_Click(object sender, EventArgs e)
{
      WCFTestService.ServiceClient myService = new
                    WCFTestService.ServiceClient();
      MessageBox.Show(myService.GetData(123));
      myService.Close();
}
Right-click the Client project and then click Set as Startup Project.
Run the Client application by pressing F5 or CTRL+F5.
When you click the button on the form, the message “You entered: 123” should appear.
Additional Resources
For more information on how to work with temporary certificates, see How to: Create Temporary Certificates for Use During Development.
For more information on how to view certificates using the Microsoft Management Console (MMC) snap in, see How to: View Certificates with the MMC Snap-in.
For more information on differences in certificate validation between Microsoft Internet Explorer and WCF, see Differences Between Service Certificate Validation Done by Internet Explorer and WCF.
For more information on differences in certificate validation between protocols, see Certificate Validation Differences Between HTTPS, SSL over TCP, and SOAP Security.
