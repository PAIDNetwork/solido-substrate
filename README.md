  # solido-substrate
  Solido v2 - An EVM / Substrate contract abstraction

  ## Introduction

  Client RPC tooling with Substrate is lacking interop with other pallets, eg EVM compatible pallets. In this use case, a potential improvement is to abstract these specific RPC client definitions into an agnostic client API.

  From previous work done by the author with [Solido](https://github.com/decent-bet/solido/), we propose to:

  - Add Ink! Smart Contracts and/or PolkadotJS provider interface
  - Integrate a subset or parts of Rosetta oriented towards client RPC scenarios
  - Add missing wallet and signing interfaces
  - Simplify event subscription using reactive patterns
  - Include support for user interface to smart contracts data binding


  ### **Add Ink! Smart Contracts and/or PolkadotJS provider interface.**

  From the previous work carried out in [Solido](https://github.com/decent-bet/solido/), as reflected in the github repository indicated in the introduction, the ability to integrate different providers widely used in the solidity smart contract ecosystem such as web3.js and Ether.js can be appreciated. in addition to provider injected wallet as metamask for ETHEREUM and connex for VeChain, in addition to taking into account that the @polkadot/api-contract interfaces provide a thin layer on the available API transactions to allow you to manage the substrate contracts in a consistent way.

  The possibility of developing a connector [Solido](https://github.com/decent-bet/solido/) to substrate.io/polkaDOT is envisaged that allows a simple, elegant and organized management of multiple interfaces to multiple providers from a single dapp, the latter having the ability to interact with multiple smart contracts in different networks. at the same time, thus achieving flexibility for developers such that it allows a smooth integration and/or migration of their applications from the ethereum environment to the substrate / PolkaDOT environment in a simple and elegant way within a code developed in typescript, bringing with it all the advantages that this entails.

  This would dramatically speed up the implementation of! Ink as the language for smart contract development on substrato.io, thus allowing early adoption of these platforms in the smart contract production environment, almost totally dominated by robustness at the moment. Mainly due to the fact that there are very few quick and easy tools to implement for Dapp developers on the substrate.io/polkaDOT platform.

  ### **Integrate a subset or parts of Rosetta oriented towards client RPC scenarios**

  In a first stage, a simple scheme of standard methods for handling dapp is foreseen that allows the integration of the interface provided by the polkadot{.js} library from it to generate a set of already predefined methods that are compatible with almost any industry standard provider like web3.js or ether.js, this can be seen in the following example:

  ```
  // Setup contract mappings
  const contractMappings = [
    {
      name: 'polkaDOT',
      import: {
        raw: {
          abi: require('./build/contracts/MetaCoin').abi,
        },
        address: {
          'local': '1nUC7afqmo7zwRFWxDjrUQu9skk6fk99pafb4SiyGSRc8z3'
        }
      },
      provider: api-contract,
      enableDynamicStubs: true,
    },
  ];

  // Create Solido Module
  const solido = new SolidoModule(contractMappings);
  const provider = new WsProvider('ws://127.0.0.1:9944');
  const PRIVATE_KEY = '...';
  const ACCOUNT = '5GrpknVvGGrGH3EFuURXeMrWHvbpj3VfER1oX5jFtuGbfzCE';
  ```

  Additionally, the solid form handles a series of predefined methods that drastically simplifies and unifies the interaction with smart contracts, both for the methods that query values ​​in said smart contracts, as well as for the methods that alter values ​​in said smart contracts and therefore expect payment on those transactions, as listed below:

  ```
  export class polkDOT.js extends SolidoProvider implements SolidoContract {
  private ws: WsProvider;
  public network: string;
  private instance: any;
  public defaultAccount: string;
  public address: string;
  private privateKey: string;

  public getProviderType(): SolidoProviderType {
    return SolidoProviderType.polka;
  }

  onReady<T>(settings: T & PolkaSettings) {
    const { privateKey, ws, network, defaultAccount } = settings;
    this.privateKey = privateKey;
    this.ws = ws;
    this.network = network;
    this.defaultAccount = defaultAccount;
    this.connect();
  }
  public connect() {
    ...
  }

  public setInstanceOptions(settings: ProviderInstance) {
    ...
  }

  async prepareSigning(
    methodCall: any,
    options: IMethodOrEventCall,
    args: any[]
  ): Promise<SolidoSigner> {
    ...
  }

  getAbiMethod(name: string): object {
    ...
  }

  callMethod(name: string, args: any[]): any {
    ...
  }
  /**
   * Gets a Ws Method object
   * @param name method name
   */
  getMethod(name: string): any {
    return this.instance.methods[name];
  }

  /**
   * Gets a provider Event object
   * @param address contract address
   * @param eventAbi event ABI
   */
  getEvent(name: string): any {
    return this.instance.events[name];
  }

  public async getEvents<P, T>(
    name: string,
    eventFilter?: EventFilter<T & any>
  ): Promise<(P)[]> {
    ...
  }
  ```

  In a second stage, the development of a structure based on standard schemas for the blockchain ecosystem that allows at the rpc level is foreseen, such that it allows calls to rpc functions from a standard predefined command system, completely abstracting the RPC layer from the blockchain facing the dapp and allowing a smooth migration, for example, from a blockchain like Ethereum to one like PolkaDOT / Substrate, without the dapp code being affected almost at all by said migration, significantly accelerating the migration dapp from solidity / ethereum environment to! ink / substrate.io. all this within the vision of Rosseta-api.org as an open standard designed to simplify the deployment and interaction with a blockchain, abstracting for the user the depth details of said particular implementation presenting a standard interface for multiple scenarios, platform and blockchain and interconnection providers to those blockchains.

  ### **Add missing wallet and signing interfaces**

  Another important feature that can be handled within solid form is the ability to integrate additional capabilities to connectors or interface providers such as web3 or ether, for example integrating solutions such as XDV, which is a multi-signature wallet that allows the handling of signing interfaces that It does not natively handle a certain wallet or dapp, it can be managed from a solid interface integrating these characteristics, the dapp with a clean and standard interface.

  ### **Simplify event subscription using reactive patterns**

  Another important feature that can be handled within solido form is the ability to handle events through industry-proven design patterns that are difficult to implement many times for developers such as the observables design pattern, a reactive event handling pattern. , ideal for managing smart contract events.

  All this within a clean interface, based on typescript that is a strongly typed language, which in turn simplifies the debugging work of the developers of said dapp, at the moment of precisely identifying the different types of data that events can drive within a smart contract.

  ```
  /***
     * Dispatches a reactive subscription for an event
     * @returns A cancellable ticket
     */
    dispatchEvent(name: string, filter: any[]): () => {} {
        let cancellable = null;
        const mapEvent = this.store.mapEvents[name];
        if (mapEvent) {
            const evt = this.instance.filters[name](...filter);
            const cb = async (...args) => {
                let mutation: any = mapEvent.mutation;
                if (typeof mapEvent.mutation === 'string') {
                    mutation = this.store.mutations[mapEvent.mutation];
                }
                try {
                    const mutateRes = await mutation([...args], this).toPromise();
                    this._subscriber.next({
                        ...this.store.state,
                        [mapEvent.getter]: mutateRes,
                    });
                } catch (e) {
                    console.log('mutation error');
                }
            };
            this.instance.on(evt, cb);
            cancellable = () => {
                this.instance.removeListener(evt, cb);
            }
        }

        return cancellable;
    }
  ```
  

  And having the entire Rx.JS operators for managing events, a number of options are available from solid to event management, subscribe and unsubscribe to an event or multiples events, with multiple functions available.

  ### **Include support for user interface to smart contracts data binding**

  As explained, solid is an entity mapper that scores a contract for Solidity based on its generated ABI. Once a contract is annotated with decorators or auto-generated, you can enable it on a blockchain using a plugin provider.

  Allowing to handle multiple scenarios, which includes a clean and elegant user interface from solid, allowing a clear simple development completely abstracting the interconnection layer of the dapp with the smart contract, making a data binding of said smart contract, in addition to allowing a *lazy loading* of one or multiple smart contracts in a dynamic way with a single interface that can be unstructured within the code, taking advantage of the benefits of typescript, handling contracts and their methods in an agile and simplified way. And regardless of the blockchain or provider that supports your connection.

  
  ```
  // Create Solido Module
  export const module = new SolidoModule(
    [
      {
        name: 'EtherToken',
        import: EnergyContractImport,
        entity: EnergyTokenContract,
        provider: Web3Plugin,
      }
    ],
  );

  module.addContractMapping({
        name: 'polkadotToken',
        import: EnergyContractImport,
        *enableDynamicStubs: true*,
        provider: polkadotPlugin,
  });

  const privateKey = '0x............';
  const chainTag = '0x4a';
  const defaultAccount = '0x...........';
  const ws = 'ws://127.0.0.1:9944';

  // polkaDOT
  const provider = new WsProvider('ws://127.0.0.1:9944');

  // polkaDOT interact with the deployed contract
  const contract = new ContractPromise(api, abi, address);

  const contracts = module.bindContracts({
    'metamask': {
      provider: metamask,
      options: {
        defaultAccount,
        chainTag,
        // ...metamask options
      }
    },
    'polkadot': {
      provider: polka,
      {
        privateKey,
        defaultAccount,
        chainTag      
      }
    }
  });

  // Add new contract and rebind 
  module.addContractMapping({
        name: 'AnotherToken',
        import: AnotherToken,
        *enableDynamicStubs: true*,
        provider: Web3Plugin,
  });

  module.rebind();

  // Get single contract
  const token = contracts.getContract<EnergyTokenContract>('polkadotToken');
  token.connect();

  // Get all contracts
  interface MyContracts {
    polkadotToken: EnergyTokenContract,
    EtherToken: EnergyTokenContract,
  }
  const { polkadotToken, EtherToken }: MyContracts = contracts.connect();

  (async () => {
    const balance = await token.balanceOf(defaultAccount);
    console.log(balance);
  })();
  ```
