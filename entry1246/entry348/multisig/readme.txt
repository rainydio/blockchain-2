1. Usage.


1.1. Create private keys, collect public keys.

Use /key.fif. Open termial, run command

  <build>/crypto/fift -I <src>/crypto/fift/lib -s key.fif <filename-base>

where <filename-base> is the name of the existing .pk file without extension. If this file does not exists, it will be created. For example

  ../build/crypto/fift -I ../src/crypto/fift/lib -s key.fif tests/wallets/0

Repeat this step for each key, and save all of the public keys.


1.2. Create the wallet.

Use /create.fif. Run command

  <build>/crypto/fift -I <src>/crypto/fift/lib -s create.fif <workchain-id> <k> <public-key-0> [...<public-key-n>]

where <public-key-0> ... <public-key-n> is the public keys from the previous step. For example

  ../build/crypto/fift -I ../src/crypto/fift/lib -s create.fif 0 2 71321937339207315216104512986610159878091710505727212000647474584844399853668 71321937339207315216104512986610159878091710505727212000647474584844399853667 71321937339207315216104512986610159878091710505727212000647474584844399853666

This will create 2-of-3 multisig wallet file /wallet.boc and addres file /wallet.addr. Wallet address can be accessed later with command

  <build>/crypto/fift -I <src>/crypto/fift/lib -s <src>/crypto/smartcont/show-addr.fif <filename-base>

For example
  ../build/crypto/fift -I ../src/crypto/fift/lib -s ../src/crypto/smartcont/show-addr.fif wallet

Get some grams on the non-bouncable address. Run the client and send file /wallet.boc

  sendfile wallet.boc


1.3. Create transaction.

Use /tx.fif. Run command

  <build>/crypto/fift -I <src>/crypto/fift/lib -s tx.fif <filename-base> <multisig-addr> <dest-addr> <seqno> <amount> <exp_time>

where <filename-base> is the name of the existing .pk file without extension, <amount> is the value in gram. For example

  ../build/crypto/fift -I ../src/crypto/fift/lib -s tx.fif tests/wallets/0 0:52a3775855ce747e626ef82b9a3ec05895132c483d1aa166bcb500d1cfb658b9 kQBSo3dYVc50fmJu-CuaPsBYlRMsSD0aoWa8tQDRz7ZYubMD 0 0.14 2000

This will create file /tx.boc which can be use immediatly or after addition some extra signatures.


1.4. Add extra signatures.

Transfer /tx.boc file to the owner of the different private key. Use /view.fif for get more details. Run command

  <build>/crypto/fift -I <src>/crypto/fift/lib -s view.fif

If the file /tx.boc exists it will print all details of this order. Use /add.fif for add one more signature. Run command

  <build>/crypto/fift -I <src>/crypto/fift/lib -s add.fif <filename-base>

where <filename-base> is the name of the existing .pk file without extension. If the file /tx.boc exists it will be extended with the new signature. For example

  ../build/crypto/fift -I ../src/crypto/fift/lib -s add.fif tests/wallets/1

Repeat this step for each signature. Run the client and send file /tx.boc

  sendfile tx.boc


1.5. View details in the blockchain.

Run the client. View seqno

  runmethod <address> seqno

View k

  runmethod <address> getk

View n

  runmethod <address> getn

View public key

  runmethod <address> getkey <key_n>

View number of the orders

  runmethod <address> getordersn

View expiration time of the order

  runmethod <address> getexptime <order_n>

View signatures of the order

  runmethod <address> getsignatures <order_n>

View hash of the order

  runmethod <address> gethash <order_n>



2. Func source code.

File /contract.fc contains funC source code.
File /contract.fif contains fift tvm assembler code generated from /contract.fc with the command

    <build>/crypto/func -o contract.fif <source>/crypto/smartcont/stdlib.fc contract.fc



3. Data structure.


3.1. Data c4.

root
  seqno, uint32
  k, uint8
  keys, ref, recursive
  orders, ref, recursive, if any

  keys
    publick key, uint256
    keys, ref, recursive, if any

  orders
    exptime, uint32
    signatures, uint128, each 1 bit corresponds to publick key
    target message hash, uin256
    orders, ref, recursive, if any


3.2. Input message.

root
  flag, uint4, set to zero for constructor
  signed message, ref
  signatures, ref, recursive

  signed message
    seqno, uint32
    exptime, uint32
    target message, ref, this will be send

  signatures
    public key, uint256
    signature, uint512
    signatures, ref, recursive



4. Tests.

Folder /tests.
Address 0:086e14f5fd61b38182b979f2ec4174d5de59297c7c1ae7b0e6553b273d1569b4, 3-of-7 wallet
Address 0:52a3775855ce747e626ef82b9a3ec05895132c483d1aa166bcb500d1cfb658b9, 2-of-7 wallet



5. Found issues.


5.1.

Incorrect parse and/or display of the short addresses, for example:
0:086e14f5fd61b38182b979f2ec4174d5de59297c7c1ae7b0e6553b273d1569b4 ->
kQAIbhT1_WGzgYK5efLsQXTV3lkpfHwa57DmVTsnPRVptBry ->
0:86e14f5fd61b38182b979f2ec4174d5de59297c7c1ae7b0e6553b273d1569b4 ->
exception


5.2.

s17 push code generated but not compiled.



6. ... to be continued
