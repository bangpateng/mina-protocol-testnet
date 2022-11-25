# Tutorial Mina Protocol Testnet Berincentivized

<p style="font-size:14px" align="right">
<a href="https://t.me/bangpateng_group" target="_blank">Join Telegram Bang Pateng<img src="https://user-images.githubusercontent.com/50621007/183283867-56b4d69f-bc6e-4939-b00a-72aa019d1aea.png" width="30"/></a>
</p>

<p align="center">
  <img height="auto" width="auto" src="https://user-images.githubusercontent.com/38981255/203963845-a591e37b-5c2d-4c20-8f32-94db19c36a05.jpg">
</p>

## Referensi

[Dokumen resmi](https://docs.minaprotocol.com/zkapps/tutorials/zkapp-ui-with-react)

[Server Discord ](https://discord.gg/zJkqXjm8)

[Install Wallet](https://www.aurowallet.com/)

[Mina Faucet](https://faucet.minaprotocol.com/)

[Rules Reward](https://minaprotocol.com/blog/zkspark-cohort0?_hsenc=p2ANqtz-8smwqFrO-bZbm3_8-KWLkOJEV5_-yyWKkPzNswcOViTtGGAsJ2Ixg_W6Efo0kaIah9zr_wPl3trIgYeeJwCA40SGbKOQ&_hsmi=234896730)

## Persyaratan hardware & software

### Persyaratan software/OS

| Komponen | Spesifikasi minimal |
|----------|---------------------|
|Sistem Operasi|Ubuntu 16.04|

| Komponen | Spesifikasi rekomendasi |
|----------|---------------------|
|Sistem Operasi|Ubuntu 20.04|

## Bahan Awal Yang Harus di Siapkan

1 . [Install Wallet](https://www.aurowallet.com/) Buat Akun dan Save Pharse Dan Ganti Jaringan ke Berkeley

2 . [Mina Faucet](https://faucet.minaprotocol.com/) Claim Faucet Masukan Address Mina Yang Kalian Dapatkan di Wallet Yang Telah Kalian Buat

## 1. Open Port

```
ufw allow 22 && ufw allow 3000
ufw enable
```

## 2. Instal Node Js 16 & NPM & Git

```
sudo apt update
sudo apt upgrade
sudo apt install git
sudo apt install -y curl
```
```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
```
```
sudo apt install -y nodejs
```

Check Versi Node Js
```
node -v
```
Check Versi NPM 
```
npm -v
```
Check Versi Git
```
git --version
```

## 3 . Install Zkapp CLI

```
git clone https://github.com/o1-labs/zkapp-cli
```
```
npm instal -g zkapp-cli@0.5.3
```

Check Version Zk Apakah Sudah Terinstall Apa Belum
```
zk --version
```

## 4 . Create Project

```
zk project 04-zkapp-browser-ui --ui next
```

1 . pilih Yes 3x dan `Enter` Biarkan Hingga Proses Instalisasi Selesai

## 5 . Buat Repository di Github kalian

Beri Nama Samain Kasih Nama `04-zkapp-browser-ui` Setelah Selesai Membuat Repo Anda Akan Mendapatkan Command di Github Diemin, Nanti Akan di Butuhkan

Balik ke VPS Kalian Jalankan : 

```
cd 04-zkapp-browser-ui
```
```
git remote add origin <your-repo-url>
```
```
git push -u origin main
```

`<Your-Repo-Url>` = Check di Repository Yang Sudah Kalian Buat

Lalu Nanti Anda Akan di Mintai Username dan Juga Password Github Anda (Jika Terjadi Error) Anda Harus Membuat Token Acces Sebagian Pengganti Password Nya, Caranya : 

1. Pergi ke `Settingan Profil` Github Anda
2. `Depelover Setting`
3. `Personal Acces Token` > Pilih yang `Token (Classic)`
4. Lalu `Generate` dan Centang Bagian `Repo`
5. Token Acces Sudah Jadi, Itu Untuk Pengganti Password Github Anda

## 6 . Run Contract

```
cd
cd 04-zkapp-browser-ui/contracts/
npm run build
```

##  7. Membuat 2 File Baru 

```
cd
cd 04-zkapp-browser-ui/ui/pages
```

### Buat 2 File Baru Berisi :

##### Folder Pertama : 
```
nano zkappWorker.ts
```
Copy dan Paste Isian Script Ini di Terminal Vps Kalian :

```
import {
    Mina,
    isReady,
    PublicKey,
    PrivateKey,
    Field,
    fetchAccount,
} from 'snarkyjs'

type Transaction = Awaited<ReturnType<typeof Mina.transaction>>;

// ---------------------------------------------------------------------------------------

import type { Add } from '../../contracts/src/Add';

const state = {
    Add: null as null | typeof Add,
    zkapp: null as null | Add,
    transaction: null as null | Transaction,
}

// ---------------------------------------------------------------------------------------

const functions = {
    loadSnarkyJS: async (args: {}) => {
        await isReady;
    },
    setActiveInstanceToBerkeley: async (args: {}) => {
        const Berkeley = Mina.BerkeleyQANet(
            "https://proxy.berkeley.minaexplorer.com/graphql"
        );
        Mina.setActiveInstance(Berkeley);
    },
    loadContract: async (args: {}) => {
        const { Add } = await import('../../contracts/build/src/Add.js');
        state.Add = Add;
    },
    compileContract: async (args: {}) => {
        await state.Add!.compile();
    },
    fetchAccount: async (args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        return await fetchAccount({ publicKey });
    },
    initZkappInstance: async (args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        state.zkapp = new state.Add!(publicKey);
    },
    getNum: async (args: {}) => {
        const currentNum = await state.zkapp!.num.get();
        return JSON.stringify(currentNum.toJSON());
    },
    createUpdateTransaction: async (args: {}) => {
        const transaction = await Mina.transaction(() => {
            state.zkapp!.update();
        }
        );
        state.transaction = transaction;
    },
    proveUpdateTransaction: async (args: {}) => {
        await state.transaction!.prove();
    },
    getTransactionJSON: async (args: {}) => {
        return state.transaction!.toJSON();
    },
};

// ---------------------------------------------------------------------------------------

export type WorkerFunctions = keyof typeof functions;

export type ZkappWorkerRequest = {
    id: number,
    fn: WorkerFunctions,
    args: any
}

export type ZkappWorkerReponse = {
    id: number,
    data: any
}
if (process.browser) {
    addEventListener('message', async (event: MessageEvent<ZkappWorkerRequest>) => {
        const returnData = await functions[event.data.fn](event.data.args);

        const message: ZkappWorkerReponse = {
            id: event.data.id,
            data: returnData,
        }
        postMessage(message)
    });
}
```

Simpan `CTRL` `X` `Y` dan `Enter`

##### Folder Kedua :

```
nano zkappWorkerClient.ts
```

Copy dan Paste Isian Script Ini di Terminal Vps Kalian :

```
import {
    fetchAccount,
    PublicKey,
    PrivateKey,
    Field,
} from 'snarkyjs'

import type { ZkappWorkerRequest, ZkappWorkerReponse, WorkerFunctions } from './zkappWorker';

export default class ZkappWorkerClient {

    // ---------------------------------------------------------------------------------------

    loadSnarkyJS() {
        return this._call('loadSnarkyJS', {});
    }

    setActiveInstanceToBerkeley() {
        return this._call('setActiveInstanceToBerkeley', {});
    }

    loadContract() {
        return this._call('loadContract', {});
    }

    compileContract() {
        return this._call('compileContract', {});
    }

    fetchAccount({ publicKey }: { publicKey: PublicKey }): ReturnType<typeof fetchAccount> {
        const result = this._call('fetchAccount', { publicKey58: publicKey.toBase58() });
        return (result as ReturnType<typeof fetchAccount>);
    }

    initZkappInstance(publicKey: PublicKey) {
        return this._call('initZkappInstance', { publicKey58: publicKey.toBase58() });
    }

    async getNum(): Promise<Field> {
        const result = await this._call('getNum', {});
        return Field.fromJSON(JSON.parse(result as string));
    }

    createUpdateTransaction() {
        return this._call('createUpdateTransaction', {});
    }

    proveUpdateTransaction() {
        return this._call('proveUpdateTransaction', {});
    }

    async getTransactionJSON() {
        const result = await this._call('getTransactionJSON', {});
        return result;
    }

    // ---------------------------------------------------------------------------------------

    worker: Worker;

    promises: { [id: number]: { resolve: (res: any) => void, reject: (err: any) => void } };

    nextId: number;

    constructor() {
        this.worker = new Worker(new URL('./zkappWorker.ts', import.meta.url))
        this.promises = {};
        this.nextId = 0;

        this.worker.onmessage = (event: MessageEvent<ZkappWorkerReponse>) => {
            this.promises[event.data.id].resolve(event.data.data);
            delete this.promises[event.data.id];
        };
    }

    _call(fn: WorkerFunctions, args: any) {
        return new Promise((resolve, reject) => {
            this.promises[this.nextId] = { resolve, reject }

            const message: ZkappWorkerRequest = {
                id: this.nextId,
                fn,
                args,
            };

            this.worker.postMessage(message);

            this.nextId++;
        });
    }
}
```

Simpan `CTRL` `X` `Y` dan `Enter`

## 8 . Tambahkan Css

```
cd
cd 04-zkapp-browser-ui/ui/styles
```

Lalu

```
nano globals.css
```

Hapus Semu Isi Yang Ada di Sana dan Ganti Dengan Script di Bawah Ini :

```
html,
body {
  padding: 0;
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen,
    Ubuntu, Cantarell, Fira Sans, Droid Sans, Helvetica Neue, sans-serif;
}

a {
  color: inherit;
  text-decoration: none;
}

* {
  box-sizing: border-box;
}

@media (prefers-color-scheme: dark) {
  html {
    color-scheme: dark;
  }
  body {
    color: white;
    background: black;
  }
}
```

Simpan `CTRL` `X` `Y` dan `Enter`

## 10 .  Running Web Buka 2 Terminal Baru (1 Perintah Masing Masing 1 Tab)

##### TAB 1
```
cd
cd 04-zkapp-browser-ui/ui/
npm run dev
```

##### TAB 2

```
cd
cd 04-zkapp-browser-ui/ui/
npm run ts-watch
```

1. Lalu Buka Browser Chrome http://ip-vps-kalian:3000
2. Pastikan Snarkjs Terbuka (Walaupun Error Abaikan Saja)
3. Lalu 2 Tab Tersebut Jika Sudah Jalan, Bisa Kalian Close Langsung Atau Kalian Bisa Menjalankan Tab Baru

## 11 . Implementing the react app

```
cd
cd 04-zkapp-browser-ui/ui/pages
nano _app.page.tsx
```

Hapus Command Yang Ada di VPS kalian Ganti Dengan Script di Bawah ini :

```
// import '../styles/globals.css'
// import type { AppProps } from 'next/app'

// import './reactCOIServiceWorker';

// export default function App({ Component, pageProps }: AppProps) {
//   return <Component {...pageProps} />
// }


import '../styles/globals.css'
import { useEffect, useState } from "react";
import './reactCOIServiceWorker';

import ZkappWorkerClient from './zkappWorkerClient';

import {
  PublicKey,
  PrivateKey,
  Field,
} from 'snarkyjs'

let transactionFee = 0.1;

export default function App() {

  let [state, setState] = useState({
    zkappWorkerClient: null as null | ZkappWorkerClient,
    hasWallet: null as null | boolean,
    hasBeenSetup: false,
    accountExists: false,
    currentNum: null as null | Field,
    publicKey: null as null | PublicKey,
    zkappPublicKey: null as null | PublicKey,
    creatingTransaction: false,
  });

  // -------------------------------------------------------
  // Do Setup

  useEffect(() => {
    (async () => {
      if (!state.hasBeenSetup) {
        const zkappWorkerClient = new ZkappWorkerClient();

        console.log('Loading SnarkyJS...');
        await zkappWorkerClient.loadSnarkyJS();
        console.log('done');

        await zkappWorkerClient.setActiveInstanceToBerkeley();

        const mina = (window as any).mina;

        if (mina == null) {
          setState({ ...state, hasWallet: false });
          return;
        }

        const publicKeyBase58: string = (await mina.requestAccounts())[0];
        const publicKey = PublicKey.fromBase58(publicKeyBase58);

        console.log('using key', publicKey.toBase58());

        console.log('checking if account exists...');
        const res = await zkappWorkerClient.fetchAccount({ publicKey: publicKey! });
        const accountExists = res.error == null;

        await zkappWorkerClient.loadContract();

        console.log('compiling zkApp');
        await zkappWorkerClient.compileContract();
        console.log('zkApp compiled');

        const zkappPublicKey = PublicKey.fromBase58('B62qrDe16LotjQhPRMwG12xZ8Yf5ES8ehNzZ25toJV28tE9FmeGq23A');

        await zkappWorkerClient.initZkappInstance(zkappPublicKey);

        console.log('getting zkApp state...');
        await zkappWorkerClient.fetchAccount({ publicKey: zkappPublicKey })
        const currentNum = await zkappWorkerClient.getNum();
        console.log('current state:', currentNum.toString());

        setState({
          ...state,
          zkappWorkerClient,
          hasWallet: true,
          hasBeenSetup: true,
          publicKey,
          zkappPublicKey,
          accountExists,
          currentNum
        });
      }
    })();
  }, []);

  // -------------------------------------------------------
  // Wait for account to exist, if it didn't

  useEffect(() => {
    (async () => {
      if (state.hasBeenSetup && !state.accountExists) {
        for (; ;) {
          console.log('checking if account exists...');
          const res = await state.zkappWorkerClient!.fetchAccount({ publicKey: state.publicKey! })
          const accountExists = res.error == null;
          if (accountExists) {
            break;
          }
          await new Promise((resolve) => setTimeout(resolve, 5000));
        }
        setState({ ...state, accountExists: true });
      }
    })();
  }, [state.hasBeenSetup]);

  // -------------------------------------------------------
  // Send a transaction

  const onSendTransaction = async () => {
    setState({ ...state, creatingTransaction: true });
    console.log('sending a transaction...');

    await state.zkappWorkerClient!.fetchAccount({ publicKey: state.publicKey! });

    await state.zkappWorkerClient!.createUpdateTransaction();

    console.log('creating proof...');
    await state.zkappWorkerClient!.proveUpdateTransaction();

    console.log('getting Transaction JSON...');
    const transactionJSON = await state.zkappWorkerClient!.getTransactionJSON()

    console.log('requesting send transaction...');
    const { hash } = await (window as any).mina.sendTransaction({
      transaction: transactionJSON,
      feePayer: {
        fee: transactionFee,
        memo: '',
      },
    });

    console.log(
      'See transaction at https://berkeley.minaexplorer.com/transaction/' + hash
    );

    setState({ ...state, creatingTransaction: false });
  }

  // -------------------------------------------------------
  // Refresh the current state

  const onRefreshCurrentNum = async () => {
    console.log('getting zkApp state...');
    await state.zkappWorkerClient!.fetchAccount({ publicKey: state.zkappPublicKey! })
    const currentNum = await state.zkappWorkerClient!.getNum();
    console.log('current state:', currentNum.toString());

    setState({ ...state, currentNum });
  }

  // -------------------------------------------------------
  // Create UI elements

  let hasWallet;
  if (state.hasWallet != null && !state.hasWallet) {
    const auroLink = 'https://www.aurowallet.com/';
    const auroLinkElem = <a href={auroLink} target="_blank" rel="noreferrer"> [Link] </a>
    hasWallet = <div> Could not find a wallet. Install Auro wallet here: {auroLinkElem}</div>
  }

  let setupText = state.hasBeenSetup ? 'SnarkyJS Ready' : 'Setting up SnarkyJS...';
  let setup = <div> {setupText} {hasWallet}</div>

  let accountDoesNotExist;
  if (state.hasBeenSetup && !state.accountExists) {
    const faucetLink = "https://faucet.minaprotocol.com/?address=" + state.publicKey!.toBase58();
    accountDoesNotExist = <div>
      Account does not exist. Please visit the faucet to fund this account
      <a href={faucetLink} target="_blank" rel="noreferrer"> [Link] </a>
    </div>
  }

  let mainContent;
  if (state.hasBeenSetup && state.accountExists) {
    mainContent = <div>
      <button onClick={onSendTransaction} disabled={state.creatingTransaction}> Send Transaction </button>
      <div> Current Number in zkApp: {state.currentNum!.toString()} </div>
      <button onClick={onRefreshCurrentNum}> Get Latest State </button>
    </div>
  }

  return <div>
    {setup}
    {accountDoesNotExist}
    {mainContent}
  </div>
}
```

## 12 . Deploy UI ke Repository Kalian

```
cd 04-zkapp-browser-ui/ui/
npm run deploy
```

Jika ada Output Error Seperti Ini Abaikan Dan Tunggu Hingga Proses Selesai :

`./pages/_app.page.tsx
95:6  Warning: React Hook useEffect has a missing dependency: 'state'. Either include it or remove the dependency array. You can also do a functional update 'setState(s => ...)' if you only need 'state' in the 'setState' call.  react-hooks/exhaustive-deps
115:6  Warning: React Hook useEffect has a missing dependency: 'state'. Either include it or remove the dependency array. You can also do a functional update 'setState(s => ...)' if you only need 'state' in the 'setState' call.  react-hooks/exhaustive-deps`

1. Nanti Ketika Selesai Anda Akan di Suruh Memasukan Username Github Kalian
2. Masukan Password Github Anda, Ingat di Awal Kita Menggunakan Token Acces. Gunakan Itu Lagi Untuk Password Github (Atau Buat Token Acces Baru Lagi)

## 13 . Untuk Memastikan Apakah Sudah Jalan Dengan Baik

1. Edit ini : `https://<Your-Username>.github.io/04-zkapp-browser-ui/index.html`
2. Your-Username = Ganti Dengan Nama Github Kalian
3. Jika Sudah, Paste ke Google Chrome
4. Jika Berjalan Dengan Baik Maka, Akan Terbuka dan Meminta Aprrove Transaksi dari Wallet Mina Kalian

## 14. Send Beberapa Transaksi Bebas Kalo Bisa 5 Lebih

1. Buka Link `https://<Your-Username>.github.io/04-zkapp-browser-ui/index.html` Kalian
2. Connect Wallet Mina nya dan Approve
3. Refresh Web Tunggu Sampe Muncul Tombol `Send Transaksi` 
4. Tunggu Muncul Pop Up Approve Transaksi
5. Emang Lama Nongol Pop Up Jadi KUDU SABAARR

## 15 . Submit Form Register

1. Link : https://fisz9c4vvzj.typeform.com/zkSparkTutorial
2. Masukan Username Discord
3. Masukan Email Kalian
4. Masukan Link Repo Yang Sudah Kalian Buat di Tahap 13
5. Masukan Link Profil Github Kalian
6. Kasih Bacotan Indah Mengenai Project Ini
7. Done
