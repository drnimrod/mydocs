# SSL Certificate Renewal Instructions for SAP BusinessObjects

This guide provides step-by-step instructions for renewing the SSL certificate for SAP BusinessObjects Enterprise XI 4.0 on a Windows server.

## Prerequisites
- Access to the server at `D:\BusinessObjects\SAP BusinessObjects Enterprise XI 4.0\win64_x64\sapjvm\bin`
- Administrative privileges to run commands and access the Central Configuration Manager
- New certificate in `.p7b` format provided by the Network Operations Center (NOC)
- Original `Keystore.keystore` file located at `D:\DigiCert\original`

## Step-by-Step Instructions

### 1. Generate a New Keystore and Certificate Signing Request (CSR)
Run the following commands in the Command Prompt from the directory:
`D:\BusinessObjects\SAP BusinessObjects Enterprise XI 4.0\win64_x64\sapjvm\bin`

```bash
keytool -genkey -dname "CN=plcreportsdev.bmc.com, OU=phx-crsdv-01, O=BMC Software Inc, L=Houston, S=Texas, C=US" -alias tomcat -keystore KeyStore.keystore -storepass changeit -keyalg RSA -validity 1000 -keysize 2048
keytool -certreq -keyalg RSA -alias tomcat -file plcreportsdev.csr -ext SAN=dns:plcreportsdev.bmc.com,dns:phx-crsdv-01.adprod.bmc.com -keystore KeyStore.keystore -validity 1000 -keysize 2048
```

### 2. Backup Existing Certificate Files
1. Create a new folder under `D:\DigiCert` named with the year the old certificate was created (e.g., `D:\DigiCert\2020`).
2. Move all files from `D:\DigiCert` to the newly created folder for backup.

### 3. Restore Original Keystore
Copy the original `Keystore.keystore` file from `D:\DigiCert\original` to `D:\DigiCert`.

### 4. Obtain and Prepare the New Certificate
1. Download the new certificate in `.p7b` format from the link provided by the NOC.
2. Copy the `.p7b` file to `D:\DigiCert`.
3. Double-click the `.p7b` file to open it in the Windows Certificate Viewer.
4. Export the root CA certificate as `root.cer` in DER format.
5. Export the intermediate certificate as `intm.cer` in DER format.
6. Save both `root.cer` and `intm.cer` to `D:\DigiCert`.

### 5. Import Certificates into the Keystore
Run the following commands from `D:\BusinessObjects\SAP BusinessObjects Enterprise XI 4.0\win64_x64\sapjvm\bin`:

```bash
keytool -import -alias root -keystore D:\DigiCert\KeyStore.keystore -trustcacerts -file D:\DigiCert\root.cer -storepass changeit
keytool -import -alias intm -keystore D:\DigiCert\KeyStore.keystore -trustcacerts -file D:\DigiCert\intm.cer -storepass changeit
keytool -import -alias tomcat -keystore D:\DigiCert\KeyStore.keystore -trustcacerts -file D:\DigiCert\plcreportsdev_bmc_com.p7b -storepass changeit
```

### 6. Verify the Keystore
1. Run the following command to list the keystore contents and save the output:
```bash
keytool -list -keystore D:\DigiCert\KeyStore.keystore -v >> D:\DigiCert\KeyStore.list
```
2. Check the size of `D:\DigiCert\KeyStore.list`. If it is approximately 12KB, the certificates have been imported correctly.

### 7. Update Tomcat with the New Certificate
1. Open the **Central Configuration Manager** from the Start menu.
2. Stop the **Server Intelligence Agent**.
3. Stop **Apache Tomcat for BI 4**.
4. Start **Apache Tomcat for BI 4**.
5. Start the **Server Intelligence Agent**.
6. Verify that the new certificate is reflected in the server URL.

## Notes
- The Tomcat server references the `D:\DigiCert\Keystore.keystore` file for certificate information, configured in `D:\BusinessObjects\tomcat\conf\server.xml` at line 66.
- Ensure all file paths and aliases match exactly as specified to avoid errors.
- If issues arise, verify the backup folder for previous certificate files or consult the NOC for assistance.