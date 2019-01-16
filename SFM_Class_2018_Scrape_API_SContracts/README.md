[<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/banner.png" width="888" alt="Visit QuantNet">](http://quantlet.de/)

## [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/qloqo.png" alt="Visit QuantNet">](http://quantlet.de/) **SFM_Class_2018_Scrape_API_SContracts** [<img src="https://github.com/QuantLet/Styleguide-and-FAQ/blob/master/pictures/QN2.png" width="60" alt="Visit QuantNet 2.0">](http://quantlet.de/)

```yaml


Name of Quantlet:  SFM_Class_2018_Scrape_API_SContracts

 

Published in:      "Clustering & Classifying Smart Contracts & SFM_Class_2018"

  

Description:       "Scrape the Etherscan API to get source code of smart contracts given the list of their hashes"

 

Keywords:          "hash, algorithm, crix, cryptocurrency, scrape, trading, fintech, Ethereum, API, Solidity"



See also :         crix, econ_arima, econ_crix, econ_garch, econ_vola



Author:            Raphael Constantin Georg Reule, Elizaveta Zinovyeva, Rui Ren, Marvin Gauer

  

Submitted:         Wed, Jan 16 2019 by Elizaveta Zinovyeva

  

Datafile:          Ethereum.csv

```

### IPYNB Code
```ipynb

{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Scrape API of etherscan to load source codes of smart contracts"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# \"Powered by Etherscan.io APIs\""
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Please place API_token for https://etherscan.io/apis into the root folder as API_token.txt \n",
    "Data set with hash addresses of contracts as Ethereum.csv in the root folder. \n",
    "Datasets structure: four columns: timestamd, hash, amount of transactions and ethereum connceted to it."
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Import Modules"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "import requests\n",
    "import pandas as pd\n",
    "import numpy as np\n",
    "import sys\n",
    "import time\n",
    "import os"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Functions"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "def split_date_year(row):\n",
    "    temp = row\n",
    "    try:\n",
    "        temp = row.split('/')[2]\n",
    "    except:\n",
    "        temp = row\n",
    "    return temp\n",
    "\n",
    "def split_date_month(row):\n",
    "    temp = row\n",
    "    try:\n",
    "        temp = row.split('/')[0]\n",
    "    except:\n",
    "        temp = row\n",
    "    return temp"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Constants and paths\n"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "full = './full/'\n",
    "sol_source = './sol_source/'\n",
    "ABI = './ABI/'\n",
    "API_Token = './API_token.txt'\n",
    "File_with_hashes = './Ethereum.csv'\n",
    "folders = [full, sol_source, ABI]\n",
    "files = [API_Token, File_with_hashes]\n",
    "develop = True # set to true if only for testing or code check"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Check whether folders exist\n",
    "##### if not create them"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "for i in folders:\n",
    "    if not os.path.exists(i):\n",
    "        os.mkdir(i)\n",
    "        \n",
    "for i in files:\n",
    "    if not os.path.exists(i):\n",
    "        print(f'file {i} does not exist')\n",
    "        print(f'Please load it and place in the root folder of the quantlet')\n",
    "    if i==API_Token:\n",
    "        f = open(f\"{API_Token}\", \"r\")\n",
    "        API_Token = f.readline()\n",
    "        f.close()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Load List of Contracts"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Load list from existing dataset\n",
    "# Datasets structure: four columns: timestamd, hash, amount of transactions and ethereum connceted to it\n",
    "contract = pd.read_csv(\"./Ethereum.csv\", delimiter=',', header=None)\n",
    "contract.dropna(axis=0, inplace=True, subset=[1])\n",
    "contract.rename(columns={0:'timestamp', 1:'ADDRESS', 2:'n_tr', 3:'ether'}, inplace=True)\n",
    "\n",
    "change = []\n",
    "for i in contract.ADDRESS:\n",
    "     change.append(i[1:])\n",
    "contract.ADDRESS = np.array(change)\n",
    "\n",
    "# Take a sample\n",
    "contract['year'] = contract.timestamp.apply(split_date_year)\n",
    "contract['month'] = contract.timestamp.apply(split_date_month)\n",
    "contract = contract[contract.year=='2018']\n",
    "contract = contract[(contract.month=='6') | (contract.month=='7')| (contract.month=='8')]\n",
    "\n",
    "address_array = contract.ADDRESS.values\n",
    "\n",
    "if develop:\n",
    "    address_array = address_array[:10]"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Function to call the API"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "def scrape_ether_contract_and_write(adress_array, API_Token):\n",
    "    full_dn = []\n",
    "    sol_source_dn = []\n",
    "    ABI_dn = []\n",
    "    c = 0\n",
    "    for i, address in enumerate(address_array):       \n",
    "        # check whether the address is already in the folder\n",
    "        if not os.path.exists(f\"{full}{address}.sol\"):\n",
    "            time.sleep(0.21) # we can do 5 GET/POST requests per sec\n",
    "            url = f'https://api.etherscan.io/api?module=contract&action=getsourcecode&address={address}&apikey={API_Token}'\n",
    "            resp = requests.get(url=url)\n",
    "            data = resp.json()\n",
    "            try:\n",
    "                # save full GET request\n",
    "                with open(f\"{full}{address}.sol\", \"w\") as d:\n",
    "                    print(data, file=d)\n",
    "                #print(f\"contract {data['result'][0]['ContractName']} downloaded\")  \n",
    "            except: \n",
    "                full_dn.append(address)\n",
    "                c += 1\n",
    "\n",
    "            try:\n",
    "                # save solidity source code\n",
    "                with open(f\"{sol_source}{address}.sol\", \"w\") as f:\n",
    "                    print(data['result'][0]['SourceCode'], file=f)\n",
    "            except: \n",
    "                sol_source_dn.append(address)\n",
    "\n",
    "            try:\n",
    "                # save ABI compiled version\n",
    "                with open(f\"{ABI}{address}.sol\", \"w\") as d:\n",
    "                    print(data['result'][0]['ABI'], file=d)\n",
    "            except: \n",
    "                ABI_dn.append(address)\n",
    "\n",
    "                if i%1000==0:\n",
    "                    print(i)\n",
    "            \n",
    "    print(f'Could not load {c} contracts')\n",
    "    return({'full_dn': full_dn, 'sol_source_dn': sol_source_dn, 'ABI_dn': ABI_dn})"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Function Call"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": [
    "dn = scrape_ether_contract_and_write(adress_array=address_array, API_Token=API_Token)"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.7"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}

```

automatically created on 2019-01-16