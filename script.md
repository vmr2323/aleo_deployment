read -p "Введіть адресу гаманця: " WALLETADDRESS
read -p "Введіть приватний ключ: " PRIVATEKEY
read -p "Введіть публічний ключ: " VIEWKEY
read -p "Введите id транзакции пополнения: " TRID
APPNAME=$(cat /dev/urandom | tr -dc 'a-z' | head -c 1 && cat /dev/urandom | tr -dc 'a-z0-9' | head -c 9)

clear
echo -e "\033[0;33mStarting...\033[0m\n"
sudo apt install -y jq build-essential pkg-config libssl-dev curl clang git gcc llvm make tmux xz-utils ufw
echo -ne "\n" | curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
git clone https://github.com/AleoHQ/leo
cd leo
cargo install --path .
cd ~
git clone https://github.com/AleoHQ/snarkOS.git --depth 1
cd snarkOS
cargo install --path .
sudo ufw allow 4133/tcp
sudo ufw allow 3033/tcp
cd
leo new "$APPNAME"
cd "$APPNAME" && leo run && cd -
PATHTOAPP=$(realpath -q $APPNAME)
cd $PATHTOAPP && cd ..
RECORD=$(curl -s "https://vm.aleo.org/api/testnet3/transaction/$TRID" | jq -r ".execution.transitions[0].outputs[0].value")
RECORD=$(snarkos developer decrypt -v $VIEWKEY -c $RECORD)
echo -e "\033[0;33mDeploying...\033[0m\n"
VAR=$(snarkos developer deploy "${APPNAME}.aleo" --private-key "${PRIVATEKEY}" --query "https://vm.aleo.org/api" --path "./${APPNAME}/build/" --broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" --fee 25000000 --record "${RECORD}")
echo $VAR
VAR=$(echo "$VAR" | tr -d '\n')
VAR=${VAR##*.}
sleep 60
RECORD=$(curl -s "https://vm.aleo.org/api/testnet3/transaction/$VAR" | jq -r ".fee.transition.outputs[0].value")
RECORD=$(snarkos developer decrypt -v $VIEWKEY -c $RECORD)
echo -e "\033[0;33mExecuting...\033[0m\n"
VAR2=$(snarkos developer execute "${APPNAME}.aleo" "main" "1u32" "2u32" --private-key "${PRIVATEKEY}" --query "https://vm.aleo.org/api" --broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" --fee 100000 --record "${RECORD}")
echo $VAR2
VAR2=$(echo "$VAR" | tr -d '\n')
VAR2=${VAR##*.}
echo -e "\033[32mDeployment finished!\033[0m\n"
echo -e "Aleo app name: \033[33m$APPNAME.aleo\033[0m\n"
echo -e "Deploy transaction ID: \033[33m$VAR\033[0m\n"
echo -e "Execute transaction ID: \033[33m$VAR2\033[0m\n"
echo -e "\033[0;33mCREATED BY ZAVOD VENTURE\033[0m"
echo -e "\033[33mTelegram: \033[36mhttps://t.me/Zavod_Venture\033[0m\n"
echo -e "\033[33mDonation:\033[0m"
echo -e "\033[36m0x439c834EE3110CeF4874E94125FE950dc8E35e2b\033[33m - ERC20\033[0m"
echo -e "\033[36mTSzyryTJkmT9fiMEVBESjA4sh9SpBw16JX\033[33m - TRC20\033[0m"
