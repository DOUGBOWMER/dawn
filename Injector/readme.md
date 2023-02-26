We know how powerful thirdweb can be, but a lot of people wonder, can I use it with Shopify? Well the answer is yes!

Whether you're an advanced Shopify theme developer or a beginner this is the guide for you.

First, if you don't already have a custom theme made, follow this guide and continue reading once you're setup - https://shopify.dev/themes/getting-started/create

Before starting, I want to define a concept we will be using throughout this guide. The "web3 island" - This means a group of React Components that must have <ThirdwebProvider /> and <ConnectWallet /> in it.

Like so:

<ThirdwebProvider>
  <ConnectWallet />
  <CustomComponent/>
</ThirdwebProvider>
Install these packages
npm i @thirdweb-dev/react @thirdweb-dev/sdk node-polyfill-webpack-plugin @babel/core @babel/preset-env @babel/preset-react babel-loader core-js react react-dom webpack webpack-cli webpack-merge
Then configure your scripts in your package.json
"scripts": {
  "dev": "webpack --config webpack.dev.js --progress --color",
  "build": "webpack --config webpack.prod.js --progress --color"
},
We are using webpack, so you need to create the following files in the root directory.
webpack.common.js
const path = require("path");
const NodePolyfillPlugin = require("node-polyfill-webpack-plugin");

module.exports = {
  entry: {
		// This is where we'll connect our react code to be bundled. Will show later
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ["babel-loader"],
      },
    ],
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "assets"),
  },
  plugins: [new NodePolyfillPlugin()],
};
webpack.dev.js
const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "development",
  devtool: "inline-source-map",
  watch: true,
});
webpack.prod.js
const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "production",
});
.babelrc
{
   "presets":[
      [
         "@babel/preset-env",
         {
            "useBuiltIns":"usage",
            "corejs":3
         }
      ],
      "@babel/preset-react"
   ]
}
Now that we have that setup, we can start writing some React components.
First we want to setup a scripts folder in the root directory, this is where we will write our React code and make sure that we are able to inject the bundled code into the liquid files.

Due to us not being able to use a provider for multiple components, we will be wrapping all of our web3 islands with it’s own provider and connect wallet button.

Let’s create our first web3 island! Start off by creating a file inside the scripts folder and call it island1.js

Here is an example of what an island could look like, I’ll walk you through each step of the way.

island1.js
import {
  ChainId,
  ConnectWallet,
  ThirdwebProvider,
  useContract,
  useNFT,
  ThirdwebNftMedia
} from "@thirdweb-dev/react";
import React from "react";
import { createRoot } from "react-dom/client";

const elements = document.querySelectorAll(".island1");

const RenderNFT = () => {
  // Get your NFT Collection using it's contract address
  const { contract } = useContract(
    "{{contractAddress}}"
  );

  // Load (and cache) the metadata for the NFT with token ID 0
  const { data: nft, isLoading } = useNFT(contract, 0);
  return !isLoading && nft ? (
    <ThirdwebNftMedia height="100px" metadata={nft.metadata} />
  ) : (
    <p>Loading...</p>
  );
};

const MyFirstWeb3Island = () => {
  return (
    <ThirdwebProvider desiredChainId={ChainId.Polygon}>
      <ConnectWallet />
      <RenderNFT />
    </ThirdwebProvider>
  );
};

elements &&
  [...elements].forEach((node) => {
    const root = createRoot(node);
    root.render(<MyFirstWeb3Island />);
  });
As you can see we grabbing all elements in our DOM with a class of island1 , we will configure this later, but keep it in mind, as you will need it in order to inject your react code at the position of that element.

const elements = document.querySelectorAll(".island1");
Now let’s breakdown the RenderNFT function here. Make sure to replace {{contractAddress}} with the contract that has your NFTs.

const RenderNFT = () => {
  // Get your NFT Collection using it's contract address
  const { contract } = useContract(
    "{{contractAddress}}"
  );

  // Load (and cache) the metadata for the NFT with token ID 0
  const { data: nft, isLoading } = useNFT(contract, 0);
  return !isLoading && nft ? (
    <ThirdwebNftMedia height="100px" metadata={nft.metadata} />
  ) : (
    <p>Loading...</p>
  );
};
First we are grabbing the contract by using thirdweb’s useContract hook. Now, we want to grab a specific NFT for that contract, so we use the useNFT hook.

Then we conditionally render an ThirdwebNftMedia component if the nft is done loading and if we found one for that tokenId.

Now we render all components together.

const MyFirstWeb3Island = () => {
  return (
    <ThirdwebProvider desiredChainId={ChainId.Polygon}>
      <ConnectWallet />
      <RenderNFT />
    </ThirdwebProvider>
  );
};
We need to ALWAYS add the ThirdwebProvider and ConnectWallet components to allow that island to pass down context/state to the components.

Lastly, we will render the component wherever you have an html tag with a class of island1 using the elements variable we saved earlier.

elements &&
  [...elements].forEach((node) => {
    const root = createRoot(node);
    root.render(<MyFirstWeb3Island />);
  });
Now we want to go to our theme.liquid file and we want to add the following script tag to the bottom of the <body>

<script src="{{ 'island1.bundle.js' | asset_url }}"></script>
Go back to webpack.common.js and add the entry for our island file. So your file should look this now.

webpack.common.js
const path = require("path");
const NodePolyfillPlugin = require("node-polyfill-webpack-plugin");

module.exports = {
  entry: {
    "island1": "./scripts/island1.js",
  },
  module: {
    rules: [
      {
        test: /\\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ["babel-loader"],
      },
    ],
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resolve(__dirname, "assets"),
  },
  plugins: [new NodePolyfillPlugin()],
};
After doing this you can run your npm script npm run dev , this will transform and bundle the react code we just wrote into the assets folder, it will create a new bundled file called island1.bundle.js

Lastly you have two options on how to position these islands. You can either create a section which will allow you to automatically place it using the Shopify theme builder or you can manually place divs with the class of island1 wherever you want.

If you chose the first option, you need to create a file under the sections folder, name it island1.liquid it should look like this.

<div class="island1"></div>

{% schema %}
{
  "name": "Island 1",
  "presets": [
    {
      "category": "React Components",
      "name": "Island 1"
    }
  ]
}
{% endschema %}
Now, this island will show up in the Shopify theme editor and you should be add it on any page / position you want! Remember before pushing your theme to your store to run npm run build .
