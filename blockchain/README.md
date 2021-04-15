# BLOCKCHAIN

## Transfer

### Transfer ETH

    public Observable<String> transferETH(String walletAddress, String privateKey, BigDecimal amount) {
        return Observable.create(emitter -> {
            Credentials credentials = Credentials.create(privateKey);
            CompletableFuture<TransactionReceipt> transactionReceiptCompletableFuture =
                Transfer.sendFunds(web3j, credentials, walletAddress,
                    amount,
                    Convert.Unit.ETHER).sendAsync();
            transactionReceiptCompletableFuture.thenAccept(transactionReceipt -> {
                Logger.d(TAG, "transferETH():transactionReceipt: " + transactionReceipt.getTransactionHash());
                emitter.onNext(transactionReceipt.getTransactionHash());
                emitter.onComplete();
            }).exceptionally(throwable -> {
                Logger.d(TAG, "transferETH(): " + throwable.getMessage());
                throwable.printStackTrace();
                emitter.onError(throwable);
                return null;
            });
        });
    }
    
### Transfer ERC20

#### Simple way

Wait transaction finish to get transaction hash

    public Observable<String> transferToken(String walletAddress, String privateKey, BigInteger amount) {
        return Observable.create(e -> {
    
            Credentials credentials = Credentials.create(privateKey);
    
            CustomToken customToken = CustomToken.load(BuildConfig.CUSTOM_TOKEN_CONTRACT_ADDRESS, web3j, credentials,
                    Convert.toWei(AppConstants.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(),
                    new BigInteger(AppConstants.GAS_LIMIT));
    
            CompletableFuture<TransactionReceipt> transactionReceiptCompletableFuture
                    = customToken.transfer(walletAddress, amount).sendAsync();
    
            transactionReceiptCompletableFuture.thenAccept(transactionReceipt -> {
                e.onNext(transactionReceipt.getTransactionHash());
                e.onComplete();
            }).exceptionally(throwable -> {
                e.onError(throwable);
                return null;
            });
        });
    }    
    
#### Transfer ERC20 with keystore

return transaction hash before transaction finish - refer Trustwallet

    public Observable<String> transferTokenByKeyStore(String walletAddress, String privateKey, BigInteger amount) {
    
        return Observable.create(e -> {
    
           createTokenTransfer(
               dataManager.getWalletAddress(),
               walletAddress,
               BuildConfig.CUSTOM_TOKEN_CONTRACT_ADDRESS,
               amount,
               Convert.toWei(AppConstants.GAS_PRICE, Convert.Unit.GWEI).toBigInteger(),
               new BigInteger(AppConstants.GAS_LIMIT)
           ).subscribe(transactionHash -> {
               Logger.d("Web3jClient", "transferTokenByKeyStore():transactionHash: " + transactionHash);
               e.onNext(transactionHash);
               e.onComplete();
           }, throwable -> e.onError(throwable));
        });
    }
    
    public Single<String> createTokenTransfer(String from, String to, String contractAddress, BigInteger amount, BigInteger gasPrice, BigInteger gasLimit) {
    
        final byte[] data = createTokenTransferData(to, amount);
    
        return Single.fromCallable(() -> {
            EthGetTransactionCount ethGetTransactionCount = web3j
                .ethGetTransactionCount(from, DefaultBlockParameterName.LATEST)
                .send();
            return ethGetTransactionCount.getTransactionCount();
        })
            .flatMap(nonce -> signTransaction(from, Telephony.Carriers.PASSWORD, contractAddress, BigInteger.valueOf(0), gasPrice, gasLimit, nonce.longValue(), data, BuildConfig.CHAIN_ID))
            .flatMap(signedMessage -> Single.fromCallable( () -> {
                EthSendTransaction raw = web3j
                    .ethSendRawTransaction(Numeric.toHexString(signedMessage))
                    .send();
                if (raw.hasError()) {
                    throw new Exception(raw.getError().getMessage());
                }
                return raw.getTransactionHash();
            })).subscribeOn(Schedulers.io());
    }
    
    public Single<byte[]> signTransaction(String signer, String signerPassword, String toAddress, BigInteger amount, BigInteger gasPrice, BigInteger gasLimit, long nonce, byte[] data, long chainId) {
        return Single.fromCallable(() -> {
            BigInt value = new BigInt(0);
            value.setString(amount.toString(), 10);
    
            BigInt gasPriceBI = new BigInt(0);
            gasPriceBI.setString(gasPrice.toString(), 10);
    
            BigInt gasLimitBI = new BigInt(0);
            gasLimitBI.setString(gasLimit.toString(), 10);
    
            Transaction tx = new Transaction(
                nonce,
                new org.ethereum.geth.Address(toAddress),
                value,
                gasLimitBI,
                gasPriceBI,
                data);
    
            BigInt chain = new BigInt(chainId); // Chain identifier of the main net
            org.ethereum.geth.Account gethAccount = findAccount(signer);
            keyStore.unlock(gethAccount, signerPassword);
            Transaction signed = keyStore.signTx(gethAccount, tx, chain);
            keyStore.lock(gethAccount.getAddress());
    
            return signed.encodeRLP();
        })
            .subscribeOn(Schedulers.io());
    }
    
    private org.ethereum.geth.Account findAccount(String address) throws Exception {
        Accounts accounts = keyStore.getAccounts();
        int len = (int) accounts.size();
        Logger.d(TAG, "findAccount(): " + len);
        for (int i = 0; i < len; i++) {
            try {
                Logger.d("ACCOUNT_FIND", "Address: " + accounts.get(i).getAddress().getHex());
                if (accounts.get(i).getAddress().getHex().equalsIgnoreCase(address)) {
                    return accounts.get(i);
                }
            } catch (Exception ex) {
                /* Quietly: interest only result, maybe next is ok. */
            }
        }
        throw new Exception("Wallet with address: " + address + " not found");
    }
    
    public static byte[] createTokenTransferData(String to, BigInteger tokenAmount) {
        List<Type> params = Arrays.<Type>asList(new Address(to), new Uint256(tokenAmount));
    
        List<TypeReference<?>> returnTypes = Arrays.<TypeReference<?>>asList(new TypeReference<Bool>() {
        });
    
        Function function = new Function("transfer", params, returnTypes);
        String encodedFunction = FunctionEncoder.encode(function);
        return Numeric.hexStringToByteArray(Numeric.cleanHexPrefix(encodedFunction));
    }
    
#### Transfer ERC20 with transactionmanager

    public void transfer() {
        Web3j web3j = Web3j.build(new HttpService("https://rinkeby.infura.io/v3/xxx"));
        Credentials credentials = Credentials.create("xxx");
    
        TransactionReceiptProcessor transactionReceiptProcessor = new NoOpProcessor(web3j);
        TransactionManager transactionManager = new RawTransactionManager(
            web3j, credentials, ChainId.RINKEBY, transactionReceiptProcessor);
    
    
        CustomToken customToken = CustomToken.load("0xxx", web3j, transactionManager, Convert.toWei("3", Convert.Unit.GWEI).toBigInteger(), new BigInteger("100000"));
    
        try {
            System.out.println("---------------------");
            TransactionReceipt transactionReceipt = customToken.transfer("0xxxx", Convert.toWei("0.01", Convert.Unit.ETHER).toBigInteger()).sendAsync().get();
            System.out.println("---------------------" + transactionReceipt.getTransactionHash());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }    
    
### Create etherium wallet

    public UserWallet createWalletWithMnemonic() {
        try {
            File keyStoreFile = new File(context.getFilesDir(), AppConstants.KEYSTORE_PATH);
    
            if (!keyStoreFile.exists()) {
                keyStoreFile.mkdirs();
            }
    
            Bip39Wallet wallet = WalletUtils.generateBip39Wallet("", keyStoreFile);
            byte[] seed = MnemonicUtils.generateSeed(wallet.getMnemonic(), "");
    
            Bip32ECKeyPair masterKeypair = Bip32ECKeyPair.generateKeyPair(seed);
            final int[] path = {44 | HARDENED_BIT, 60 | HARDENED_BIT, 0 | HARDENED_BIT, 0, 0};//web3j is wrong here, don't use Bip44WalletUtils to create
            Bip32ECKeyPair childKeypair = Bip32ECKeyPair.deriveKeyPair(masterKeypair, path);
    
            Credentials credentials = Credentials.create(childKeypair);
    
            UserWallet userWallet = new UserWallet(credentials.getAddress(),
                credentials.getEcKeyPair().getPrivateKey().toString(16),
                wallet.getMnemonic());
    
            this.generateKeystore(userWallet.getPrivateKey());
    
            return userWallet;
        } catch (Exception ex) {
            ex.printStackTrace();
            return null;
        }
    }
    
    public String generateKeystore(String privateKey) throws CipherException, IOException {
        File keyStoreFile = new File(context.getFilesDir(), AppConstants.KEYSTORE_PATH);
    
        if (!keyStoreFile.exists()) {
            keyStoreFile.mkdirs();
        }
    
        Credentials credentials = Credentials.create(privateKey);
        return WalletUtils.generateWalletFile(PASSWORD, credentials.getEcKeyPair(), keyStoreFile, false);
    }
    
### Import wallet by mnemonic

    public Credentials importWalletByMnemonic(String mnemonic) {
        byte[] seed = MnemonicUtils.generateSeed(mnemonic, "");
        Bip32ECKeyPair masterKeypair = Bip32ECKeyPair.generateKeyPair(seed);
        final int[] path = {44 | HARDENED_BIT, 60 | HARDENED_BIT, 0 | HARDENED_BIT, 0, 0};//web3j is wrong here, don't use Bip44WalletUtils to import
        Bip32ECKeyPair childKeypair = Bip32ECKeyPair.deriveKeyPair(masterKeypair, path);
        return Credentials.create(childKeypair);
    }
    
        