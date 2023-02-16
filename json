// create a trading bot on Polygon which uses Quickswap's contracts to get prices and swaps two tokens

// Imports
const ethers = require("ethers");
const secrets = require("./secrets.json");
const fetch = require("node-fetch");

const USDC = "0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174"; //mainnet
//USDC on testnet
// const USDC = "0xe11A86849d99F524cAC3E7A0Ec1241828e332C62";

const KAXAA = "0x01426ebDEFaD8C89cB748499af0E4809B77A9676"; //mainnet
// MATIC on testnet
// const KAXAA = "0x9c3C9283D3e44854697Cd22D3Faa240Cfb032889";

const quickswapRouter = "0xa5E0829CaCEd8fFDD4De3c43696c57F7D7A678ff"; //mainnet
// Quickswap router on testnet
// const quickswapRouter = "0x8954AfA98594b838bda56FE4C12a09D7739D179b";

// Connect to mainnet with an API key (these are equivalent)
const provider = new ethers.providers.AlchemyProvider(
  "matic",
  secrets.ALCHEMY_API_KEY
);

// const provider = new ethers.providers.JsonRpcProvider(secrets.provider);
const wallet = new ethers.Wallet(secrets.WALLET_PRIVATE_KEY);
const signer = wallet.connect(provider);

const quickswapRouterContract = new ethers.Contract(
  quickswapRouter,
  [
    "function getAmountsOut(uint amountIn, address[] calldata path) external view returns (uint[] memory amounts)",
    "function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts)",
  ],
  signer
);

const KAXAAContract = new ethers.Contract(
  KAXAA,
  ["function approve(address spender, uint256 amount) external returns (bool)"],
  signer
);

const USDCContract = new ethers.Contract(
  USDC,
  ["function approve(address spender, uint256 amount) external returns (bool)"],
  signer
);

async function sellKAXAATokens(KAXAATokensToSell) {
  const KAXAAamountIn = ethers.utils.parseUnits(
    KAXAATokensToSell.toString(),
    18
  );
  console.log(ethers.utils.formatEther(KAXAAamountIn));

  let amounts = await quickswapRouterContract.getAmountsOut(
    KAXAAamountIn,
    [KAXAA, USDC],
    {
      gasLimit: 3000000,
      gasPrice: 45844749388,
    }
  );

  let USDCamountOutMin = amounts[1].sub(amounts[1].div(5));
  USDCamountOutMin = ethers.utils.parseUnits("0", 6);

  console.log(amounts);

  console.log(ethers.utils.formatEther(KAXAAamountIn));
  console.log(ethers.utils.formatUnits(USDCamountOutMin, 6));

  const approveTx = await KAXAAContract.approve(
    quickswapRouter,
    KAXAAamountIn,
    {
      gasLimit: 3000000,
      gasPrice: 65844749388,
    }
  );
  let reciept = await approveTx.wait();
  console.log(reciept);

  const swapTx = await quickswapRouterContract.swapExactTokensForTokens(
    KAXAAamountIn,
    USDCamountOutMin,
    [KAXAA, USDC],
    wallet.address,
    Date.now() + 1000 * 60 * 10, // 10 minutes
    {
      gasLimit: 3000000,
      gasPrice: 55844749388,
    }
  );

  receipt = await swapTx.wait();
  console.log(receipt);
}

async function sellUSDCTokens(USDCToSell) {
  const USDCamountIn = ethers.utils.parseUnits(USDCToSell.toString(), 6);
  console.log(ethers.utils.formatUnits(USDCamountIn, 6));

  let amounts = await quickswapRouterContract.getAmountsOut(
    USDCamountIn,
    [USDC, KAXAA],
    {
      gasLimit: 3000000,
      gasPrice: 45844749388,
    }
  );

  let KAXAAamountOutMin = amounts[1].sub(amounts[1].div(5));
  KAXAAamountOutMin = ethers.utils.parseUnits("0", 18);

  console.log(amounts);

  console.log(ethers.utils.formatUnits(USDCamountIn, 6));
  console.log(ethers.utils.formatUnits(KAXAAamountOutMin, 18));

  const approveTx = await USDCContract.approve(quickswapRouter, USDCamountIn, {
    gasLimit: 3000000,
    gasPrice: 65844749388,
  });
  let reciept = await approveTx.wait();
  console.log(reciept);

  const swapTx = await quickswapRouterContract.swapExactTokensForTokens(
    USDCamountIn,
    KAXAAamountOutMin,
    [USDC, KAXAA],
    wallet.address,
    Date.now() + 1000 * 60 * 10, // 10 minutes
    {
      gasLimit: 3000000,
      gasPrice: 55844749388,
    }
  );

  receipt = await swapTx.wait();
  console.log(receipt);
}

async function main() {
  // Fetch the price of KAXAA from an API

  const response = await fetch("https://indexapi.kaxaa.com/?type=json");
  const data = await response.json();
  console.log(data["value"]);

  const KAXAAStablePrice = data["value"];

  const KAXAAamountIn = ethers.utils.parseUnits("1", 18);
  let amounts = await quickswapRouterContract.getAmountsOut(KAXAAamountIn, [
    KAXAA,
    USDC,
  ]);
  console.log(amounts);
  // Assuming slippage of 10% and USDC has 6 decimals
  let priceOfKAXAAInUSDC = amounts[1].sub(amounts[1].div(10));
  priceOfKAXAAInUSDC = ethers.utils.formatUnits(priceOfKAXAAInUSDC, 6);
  console.log(priceOfKAXAAInUSDC);
  if (KAXAAStablePrice > priceOfKAXAAInUSDC) {
    console.log("Sell USDC for KAXAA");
    await sellUSDCTokens(1);
  } else if (KAXAAStablePrice < priceOfKAXAAInUSDC) {
    console.log("Sell KAXAA for USDC");
    await sellKAXAATokens(100);
  }
}

main()
  .then()
  .finally(() => {
    console.log("Done");
  });
