# Decrypt secret key base64 by openssl

## Decrypt by command

Convert base64 to binary

        cat secretkey.txt | base64 -d > secretkey.bin
        
Decrypt by pem

        openssl rsautl -decrypt -in secretkey.bin -out out.txt -inkey privatekey.pem         

## Decrypt by java

### Get private key alias

        keytool -list -storetype pkcs12 -keystore privatekey.p12 -storepass p12_password
        keytool -v -list -storetype pkcs12 -keystore privatekey.p12 -storepass p12_password
        
## Add dependency

        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcpkix-jdk15on</artifactId>
            <version>1.68</version>
        </dependency>        

### Implement

        String ALGORITHM = "RSA";
        String p12Password = "p12_password";
        String privateKeyPath = "C:\\tmp\\privatekey.p12";
        String alias = "alias";

        byte[] inputData = Base64.decodeBase64(secretKey);

        KeyStore keystore = KeyStore.getInstance("PKCS12");
        keystore.load(new FileInputStream(privateKeyPath), p12Password.toCharArray());
        PrivateKey key = (PrivateKey) keystore.getKey(alias, p12Password.toCharArray());

        Cipher cipher = Cipher.getInstance(ALGORITHM);
        cipher.init(Cipher.DECRYPT_MODE, key);

        byte[] decryptedBytes = cipher.doFinal(inputData);