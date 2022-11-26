#04-zkapp-browser-ui



## Reference

[Official document](https://docs.minaprotocol.com/zkapps/tutorials/zkapp-ui-with-react)

[Discord Server ](https://discord.gg/zJkqXjm8)

[Install Wallet](https://www.aurowallet.com/)

[Mina Faucet](https://faucet.minaprotocol.com/)

[Reward Rules](https://minaprotocol.com/blog/zkspark-cohort0?_hsenc=p2ANqtz-8smwqFrO-bZbm3_8-KWLkOJEV5_-yyWKkPzNswcOViTtGGAsJ2Ixg_W6Efo0kaIah9w73tr_Ixg_W6Efo0kaIah9zr0)

## Hardware & software requirements

### Software/OS requirements

| Components | Minimum specifications |
|----------|---------------------|
|Operating Systems|Ubuntu 16.04|

| Components | Recommended specifications |
|----------|---------------------|
|Operating Systems|Ubuntu 20.04|

## Initial Materials That Must Be Prepared

1 . [Install Wallet](https://www.aurowallet.com/) Create Account and Save Pharse And Switch Network to Berkeley

2 . [Mina Faucet](https://faucet.minaprotocol.com/) Claim Faucet Enter the Mina Address You Get in the Wallet You Have Created

## 1. Open Port

```
ufw allow 22 && ufw allow 3000
ufw enable
```

## 2. Install Node Js 16 & NPM & Git

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

Check Node Js Version
```
node -v
```
Check NPM Version
```
npm -v
```
Check Git Version
```
git --version
```

## 3 . Install Zkapp CLI

```
git clone https://github.com/o1-labs/zkapp-cli
```
```
npm install -g zkapp-cli@0.5.3
```

Check Zk Version Is It Already Installed Or Not
```
zk --version
```

##4 . Create Project

```
zk project 04-zkapp-browser-ui --ui next
```

1 . select Yes 3x and `Enter` Leave it until the installation process is complete

##5 . Create a repository on your Github

Give the name Samain Give the name `04-zkapp-browser-ui` After you finish creating the repo you will get a command on Github Diemin, it will be needed later

Back to your VPS Run:

```
cd 04-zkapp-browser-ui
```
```
git remote add origin <your-repo-url>
```
```
git push -u origin main
```

`<Your-Repo-Url>` = Check the Repository You Have Created

Then later you will be asked for your Github username and password (if an error occurs) you must create an access token to replace the password, here's how:

1. Go to your Github `Profile Settings`
2. `Depover Settings`
3. `Personal Access Token` > Select the `Token (Classic)`
4. Then `Generate` and Tick the `Repo` . section
5. Access token is ready, it's for your Github password replacement

##6 . Run Contract

```
CD
cd 04-zkapp-browser-ui/contracts/
npm run build
```

## 7. Create 2 New Files 
```
CD
cd 04-zkapp-browser-ui/ui/pages
```

### Create 2 New Files Contains:

##### First Folder :
```
nano zkappWorker.ts
```
Copy and Paste this Script Field in your Vps Terminal:

```
import {
    Mina,
    isReady,
    publickey,
    PrivateKey,
    fields,
    fetchAccount,
} from 'snarkyjs'

type Transaction = Awaited<ReturnType<typeof Mina.transaction>>;

// ------------------------------------------------ ---------------------------------------

import type { Add } from '../../contracts/src/Add';

conststate = {
    Add: null as null | typeof Add,
    zkapp: null as null | add,
    transaction: null as null | transactions,
}

// ------------------------------------------------ ---------------------------------------

const functions = {
    loadSnarkyJS: async(args: { }) => {
        await isReady;
    },
    setActiveInstanceToBerkeley: async(args: { }) => {
        const Berkeley = Mina.BerkeleyQANet(
            "https://proxy.berkeley.minaexplorer.com/graphql"
        );
        Mina.setActiveInstance(Berkeley);
    },
    loadContract: async(args: {}) => {
        const { Add } = await import('../../contracts/build/src/Add.js');
        state.Add = Add;
    },
    compileContract: async(args: { }) => {
        await state.Add!.compile();
    },
    fetchAccount: async(args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        return await fetchAccount({ publicKey });
    },
    initZkappInstance: async(args: { publicKey58: string }) => {
        const publicKey = PublicKey.fromBase58(args.publicKey58);
        state.zkapp = new state.Add!(publicKey);
    },
    getNum: async(args: {}) => {
        const currentNum = await state.zkapp!.num.get();
        return JSON.stringify(currentNum.toJSON());
    },
    createUpdateTransaction: async(args: {}) => {
        const transaction = await Mina.transaction(() => {
            state.zkapp!.update();
        }
        );
        state.transaction = transaction;
    },
    proveUpdateTransaction: async(args: { }) => {
        await state.transaction!.prove();
    },
    getTransactionJSON: async(args: { }) => {
        return state.transaction!.toJSON();
    },
};

// ------------------------------------------------ ---------------------------------------

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

Save `CTRL` `X` `Y` and `Enter`

##### Second Folder :
```
nano zkappWorkerClient.ts
```

Copy and Paste this Script Field in your Vps Terminal:

```
import {
    fetchAccount,
    publickey,
    PrivateKey,
    fields,
} from 'snarkyjs'

import type { ZkappWorkerRequest, ZkappWorkerReponse, WorkerFunctions } from './zkappWorker';

export default class ZkappWorkerClient {

    // ------------------------------------------------ ---------------------------------------

    loadSnarkyJS() {
        return this._call('loadSnarkyJS', { });
    }

    setActiveInstanceToBerkeley() {
        return this._call('setActiveInstanceToBerkeley', { });
    }

    loadContract() {
        return this._call('loadContract', { });
    }

    compileContract() {
        return this._call('compileContract', { });
    }

    fetchAccount({ publicKey }: { publicKey: PublicKey }): ReturnType<typeof fetchAccount> {
        const result = this._call('fetchAccount', { publicKey58: publicKey.toBase58() });
        return(result as ReturnType<typeof fetchAccount>);
    }

    initZkappInstance(publicKey: PublicKey) {
        return this._call('initZkappInstance', { publicKey58: publicKey.toBase58() });
    }

    async getNum(): Promise<Field> {
        const result = await this._call('getNum', { });
        return Field.fromJSON(JSON.parse(result as string));
    }

    createUpdateTransaction() {
        return this._call('createUpdateTransaction', { });
    }

    proveUpdateTransaction() {
        return this._call('proveUpdateTransaction', { });
    }

    async getTransactionJSON() {
        const result = await this._call('getTransactionJSON', { });
        return results;
    }

    // ------------------------------------------------ ---------------------------------------

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

Save `CTRL` `X` `Y` and `Enter`

##8 . Add Css

```
CD
cd 04-zkapp-browser-ui/ui/styles
```

Then

```
nano globals.css
```

Delete all the contents there and replace it with the script below:

```
html,
body {
  padding: 0;
  margins: 0;
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
    color scheme: dark;
  }
  body {
    color: white;
    background: black;
  }
}
```

Save `CTRL` `X` `Y` and `Enter`

## 10 . Running Web Open 2 New Terminals (1 Command Each 1 Tab)
##### TABS 1
```
CD
cd 04-zkapp-browser-ui/ui/
npm run dev
```

##### TAB 2

```
CD
cd 04-zkapp-browser-ui/ui/
npm run ts-watch
```

1. Then Open Chrome Browser http://ip-vps-you:3000
2. Make sure Snarkjs is Open (Although Ignore Errors)
3. Then the 2 tabs, if they are already running, you can close immediately or you can run a new tab

##11 . Implementing the react app

```
CD
cd 04-zkapp-browser-ui/ui/pages
nano _app.page.tsx
```

Delete the existing command on your VPS, replace it with the script below:

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

##12 . Deploy UI to Your Repository

```
cd 04-zkapp-browser-ui/ui/
npm run deploy
```

If there is an output error like this, ignore it and wait until the process is complete:

`./pages/_app.page.tsx
95:6 Warning: React Hook useEffect has a missing dependency: 'state'. Either include it or remove the dependency array. You can also do a functional update 'setState(s => ...)' if you only need 'state' in the 'setState' call. react-hooks/exhaustive-deps
115:6 Warning: React Hook useEffect has a missing dependency: 'state'. Either include it or remove the dependency array. You can also do a functional update 'setState(s => ...)' if you only need 'state' in the 'setState' call. react-hooks/exhaustive-deps`

1. Later when finished you will be asked to enter your Github username
2. Enter your Github password, remember at the beginning we used the access token. Use It Again For Github Password (Or Create New Access Token Again)

##13 . To Make Sure It's Running Well

1. Edit this : `https://<Your-Username>.github.io/04-zkapp-browser-ui/index.html`
2. Your-Username = Replace with Your Github Name
3. If it is, Paste it into Google Chrome
4. If it goes well, it will open and ask for a transaction approval from your Mina Wallet

## 14. Send Some Free Transactions If You Can 5 More

1. Go to your `https://<Your-Username>.github.io/04-zkapp-browser-ui/index.html` link
2. Connect Mina's Wallet and Approve
3. Refresh the Web Wait Until the `Send Transaction` Tombol Button Appears
4. Wait for the Approve Transaction Pop Up Appears > Fill in Fee 1
5. It takes a long time for pop ups to appear, so you have to be patient

##15 . Submit Registration Form

1. Link : https://fisz9c4vvzj.typeform.com/zkSparkTutorial
2. Enter the Discord Username
3. Enter Your Email
4. Enter the Repo Link that you created in Stage 13 > Click on your Github Profile > Click on your Repository > Click on 04-zkapp-browser-ui > Enter the link in the form
5. Enter Your Github Web Link (The One You Use To Connect To Wallet)
6. Love Beautiful Quotes About This Project
7. Done

##16 . How To Uninstall All (If You Want To Delete)

```
rm -rf 04-zkapp-browser-ui
rm -rf zkapp-cli
rm -rf .npm
npm uninstall -g zkapp-cli
sudo apt-get remove nodejs
```
