<script>
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-types";
  import { TezosToolkit } from "@taquito/taquito";
  import { Navbar, NavBrand, Button, NavHamburger, Badge } from "flowbite-svelte";
  import { Card } from "flowbite-svelte";
  import { ClockOutline, DollarSolid, WalletSolid } from "flowbite-svelte-icons";
  import Time from "svelte-time/src/Time.svelte";

  const rpcUrl = "https://ghostnet.ecadinfra.com";
  const Tezos = new TezosToolkit(rpcUrl);
  const contractAddress = "KT1KutsbggvvkHCq7ypnByTx7VsVsGEfTdVf";

  let wallet;
  let address;
  let balance;

  let history = [];

  let whitelisted = false;

  let depositButtonActive = false;

  const connectWallet = async () => {
    const newWallet = new BeaconWallet({
      name: "Simple dApp tutorial",
      network: {
        type: NetworkType.GHOSTNET,
      },
    });
    await newWallet.requestPermissions();
    address = await newWallet.getPKH();
    await getWalletBalance(address);
    wallet = newWallet;
    depositButtonActive = true;
    isWhitelisted();
  };

  const isWhitelisted = async () => {
    fetch(`https://api.ghostnet.tzkt.io/v1/contracts/${contractAddress}/storage`)
    .then((response) => response.json())
      .then((data) => {
        whitelisted = data.whitelist.hasOwnProperty(address)
      })
      .catch((error) => {
        console.log(error);
        return [];
      });
  }

  const disconnectWallet = () => {
    wallet.client.clearActiveAccount();
    wallet = undefined;
  };

  const loadTransactionHistory = async () => {
    fetch(`https://api.ghostnet.tzkt.io/v1/accounts/${address}/operations`)
      .then((response) => response.json())
      .then((data) => {
        history = data;
      })
      .catch((error) => {
        console.log(error);
        return [];
      });
  };

  const getWalletBalance = async (walletAddress) => {
    const balanceMutez = await Tezos.tz.getBalance(walletAddress);
    balance = balanceMutez.div(1000000).toFormat(2);
  };

  // Get token balance
  // const getTokenBalance = async () => {
  //   fetch(`https://api.ghostnet.tzkt.io/v1/tokens/balances`)
  //     .then((response) => response.json())
  //     .then((data) => {
  //       history = data;
  //     })
  //     .catch((error) => {
  //       console.log(error);
  //       return [];
  //     });
  // }
</script>

<main>
  <Navbar color="none" class="mt-4">
    <NavBrand href="/">
      <img
        src="/src/assets/sogelife.jpg"
        class="me-3 h-6 sm:h-9"
        alt="sogelife logo"
      />
      <span
        class="self-center whitespace-nowrap text-xl font-semibold dark:text-white"
        >SOGE Fund Dashboard</span
      >
    </NavBrand>
    <div class="flex md:order-2">
      <Button size="sm" on:click={connectWallet}>Connect Wallet</Button>
      <NavHamburger />
    </div>
  </Navbar>
  <div class="container">
    {#if whitelisted}
      <Card color="gray" size="lg">
        <h5 class="mb-4 text-2xl font-bold tracking-tight text-gray-900 dark:text-white">Wallet</h5>
        <div class="flex gap-4 items-center mb-3 font-normal text-gray-500 dark:text-gray-400">
          <WalletSolid class="w-7 h-7 text-gray-500 dark:text-gray-400" />
          <span class="font-bold">{address}</span>
        </div>
        <div class="flex gap-4 items-center mb-3 font-normal text-gray-500 dark:text-gray-400">
          <DollarSolid class="w-7 h-7 text-gray-500 dark:text-gray-400" />
          <span class="font-bold">{balance} XTZ</span>
        </div>
        {#if history.length != 0}
          <div class="mb-4">
            <div class="flex items-center gap-4">
              <ClockOutline class="w-7 h-7 text-gray-500 dark:text-gray-400"/>
              <h3>Transactions</h3>
            </div>
            {#each history as { type, timestamp, hash}}
              <div class="transact-container mb-2">
                <Time class="text-xs" timestamp={timestamp} format="dddd h:mm A - MMMM D, YYYY"></Time>
                <div class="flex gap-2">
                  <Badge rounded color="purple">{type}</Badge>
                  <p class="text-sm">{hash}</p>
                </div>
              </div>
            {/each}
          </div>
        {/if}
        <div>
          {#if history.length == 0}
            <Button on:click={loadTransactionHistory}>Load transactions</Button>
          {/if}
          <Button on:click={disconnectWallet}>Disconnect wallet</Button>
        </div>
      </Card>
    {:else if wallet}
      <h1>YOUR NOT WHITELIST!</h1>
    {:else}
      <div class="flex flex-col items-center">
        <h2 class="mb-6">Please connect your wallet</h2>
        <Button size="sm" on:click={connectWallet}>Connect Wallet</Button>
      </div>
    {/if}
  </div>
</main>

<style>
  main {
    height: 100vh;
    width: 100%;
    display: flex;
    align-items: center;
    flex-direction: column;
  }
  .container {
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .transact-container {
    border-radius: 2px;
    text-overflow: ellipsis;
    overflow: hidden;
    padding: 0.25em 0.5em;
  }
</style>
