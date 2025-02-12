# The source code

The code is shown in this dir, and we suggest the usage of docker. 

### Using Docker (Suggested)

```bash
docker pull scripthub/rsymx:v3
```

### Using Git

```bash
git clone https://github.com/SmartContract/RSymX.git && cd rsymx-master
```

Note that the solc-select will use the solc bin files in the /home/user/.solc-select (if user) or /root/.solc-select (if root user), so we need to unzip the solc files to that path by executing the following command.

```bash
tar -zcvf code/sourcecodes.part01.rar rsymx
unzip -d /home/user .solc-select.zip
cd /home/user/.solc-select/bin
rm solc-default
ln -s /root/.solc-select/usr/bin/solc-v0.4.0* solc-default
```

If you want to recreate the virtual environment, perform the following operations. Note that python3.8 needs to be replaced with the user's Python version.

```bash
virtualenv --python=/usr/bin/python3.8 venv
pip install torch==1.10.2+cpu -f https://download.pytorch.org/whl/torch_stable.html
pip install tqdm
pip install scikit-learn==1.0.2
pip install pandas
pip install numpy==1.21.6
pip install pyevmasm==0.2.3
pip install evm_cfg_builder==0.3.1
pip install reportlab==3.6.12
pip install seaborn==0.12.2
pip install reportlab==3.6.12
pip install joblib==1.3.2
export PATH=$HOME/.solc-select:$PATH add to the bottom of venv/bin/activate
apt-get install graphviz (for the CFG figure generarion)
```

After that, the main install command is shown as follows.

```bash
pip install -e ".[native]"
```

Also, the solvers such as Z3, CVC4, and Yices need to be installed based on the following operations, which are the same as the Manticore.

#### Installing Z3

Using pip to install the solver Z3.

```bash
pip install zipp==3.19.2
pip install importlib_resources==6.4.0
pip install z3-solver==4.13.0.0
```

#### Installing CVC4

For more details go to https://cvc4.github.io/. Otherwise, just get the binary and use it.

```bash
sudo wget -O /usr/bin/cvc4 https://github.com/CVC4/CVC4/releases/download/1.7/cvc4-1.7-x86_64-linux-opt
sudo chmod +x /usr/bin/cvc4
```

#### Installing Yices

Yices is incredibly fast. More details here https://yices.csl.sri.com/

```bash
sudo add-apt-repository ppa:sri-csl/formal-methods
sudo apt-get update
sudo apt-get install yices2
```

### The usage of RSymX (in docker environment, importantly)

Update the current running dir (xxx can be replaced with your specific dir). Otherwise, a solc error will be obtained when executing the following commands. 

```bash
docker run -it -v xxx/data:/data scripthub/rsymx:v3 /bin/bash
cd /rsymx/rsymx-master/
```

Detect contract source code and output the detailed and simply reports. Among the following command, the meanings of parameters are script's path, contract source code's path, solc version, output dir, reports' types, path exploration maximum (None refers that all paths are searched, aiming to obtain the comprehensive CFG), agent flag (True is on, i.e., RSymX; otherwise, False is SymX), and state parallel number (1 is single process, the most apparent advantages of RSymX. and it can be replaced with another number according to the device's capability) in orderly.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_num_contracts_sourcedetection_system /rsymx/test/arbitrary_send.sol 0.4.24 /data/test22 main,all None True 1
```

Detect contract bytecode and output the detailed and simply reports. The meanings of parameters are the same with the above command, and the only difference is that the bytecode file does not require the solc vertion.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_num_contracts_binarydetection_system /rsymx/test/arbitrary_send_bin.json /data/test30-true main,all None True 1
```

Detect contrac opcode and output the detailed and simply reports. The meanings of parameters are the same with the above command.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_num_contracts_evmdetection_system /rsymx/test/arbitrary_send_evm.json /data/test32-true main,all None True 1
```

Detect contrac source code and does not output the CFG of the contract. Note that, the path exploration maximum is set at 2, and RSymX can discover the vulnerable path, yet SymX (agent flag is set with False) cannot find it. Users can set it based on their requirements and devices' capability.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_num_contracts_sourcedetection /rsymx/test/arbitrary_send.sol 0.4.24 /data/test22 2 True 1
```

Detect contrac bytecode.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_num_contracts_binarydetection /rsymx/test/arbitrary_send_bin.json /data/test27-true None True 1
```

Detect contrac opcode.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_num_contracts_evmdetection /rsymx/test/arbitrary_send_evm.json /data/test31-true None True 1
```

Identify contrac behavior with source code.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_transaction_detection_source /rsymx/test/RealOldFuckMaker.sol 0.4.24 /rsymx/test/user_0000000f.tx.json /data/RealOldFuckMaker1-2
```

Identify contrac behavior with bytecode.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_transaction_detection_binary /rsymx/test/RealOldFuckMaker_bin.json /rsymx/test/user_0000000f.tx.json /data/RealOldFuckMaker2-2
```

Identify contrac behavior with opcode.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_transaction_detection_evm /rsymx/test/RealOldFuckMaker_evm.json /rsymx/test/user_0000000f.tx.json /data/RealOldFuckMaker3-2
```

Identify contrac behavior with init transaction.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/method_testcase_transaction_detection_tx /rsymx/test/user_0000000f_init.tx.json /rsymx/test/user_0000000f.tx.json /data/RealOldFuckMaker4-2
```

Validate detection result of *arbitrary-send* vulnerability.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/arbitrary-send_transaction_verification /rsymx/test/Arbitrarysend_validation.sol 0.4.24 /rsymx/test/Arbitrarysend_validation/user_00000005.tx.json /data/Arbitrarysend-1
```

Validate detection result of *reentrancy-eth* vulnerability.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/reentrancy_transaction_verification /rsymx/test/Reentrance_validation.sol 0.4.24 /rsymx/test/Reentrance_validation/user_00000001.tx.json /data/Reentrance-1
```

Validate detection result of *suicidal* vulnerability.

```bash
/rsymx/rsymx-master/dist/method_testcase_num_contracts_sourcedetection_system/suicidal_transaction_verification /rsymx/test/Suicidal_validation.sol 0.4.24 /rsymx/test/Suicidal_validation/user_00000001.tx.json /data/Suicidal_validation-
```