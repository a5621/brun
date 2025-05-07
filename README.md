// SPDX-License-Identifier: MIT
pragma solidity ^0.8.4;

abstract contract Context {
    function _msgSender() internal view virtual returns (address payable) {
        return payable(msg.sender);
    }

    function _msgData() internal view virtual returns (bytes memory) {
        this;
        return msg.data;
    }
}

interface IERC20 {
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address recipient, uint256 amount) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint256 amount) external returns (bool);
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}

library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }

    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        return sub(a, b, "SafeMath: subtraction overflow");
    }

    function sub(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b <= a, errorMessage);
        uint256 c = a - b;
        return c;
    }

    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) return 0;
        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");
        return c;
    }

    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        return div(a, b, "SafeMath: division by zero");
    }

    function div(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b > 0, errorMessage);
        uint256 c = a / b;
        return c;
    }

    function mod(uint256 a, uint256 b) internal pure returns (uint256) {
        return mod(a, b, "SafeMath: modulo by zero");
    }

    function mod(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b != 0, errorMessage);
        return a % b;
    }
}

library Address {
    function isContract(address account) internal view returns (bool) {
        bytes32 codehash;
        bytes32 accountHash = 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;
        assembly { codehash := extcodehash(account) }
        return (codehash != accountHash && codehash != 0x0);
    }

    function sendValue(address payable recipient, uint256 amount) internal {
        require(address(this).balance >= amount, "Address: insufficient balance");
        (bool success, ) = recipient.call{ value: amount }("");
        require(success, "Address: unable to send value, recipient may have reverted");
    }

    function functionCall(address target, bytes memory data) internal returns (bytes memory) {
        return functionCall(target, data, "Address: low-level call failed");
    }

    function functionCall(address target, bytes memory data, string memory errorMessage) internal returns (bytes memory) {
        return _functionCallWithValue(target, data, 0, errorMessage);
    }

    function functionCallWithValue(address target, bytes memory data, uint256 value) internal returns (bytes memory) {
        return functionCallWithValue(target, data, value, "Address: low-level call with value failed");
    }

    function functionCallWithValue(address target, bytes memory data, uint256 value, string memory errorMessage) internal returns (bytes memory) {
        require(address(this).balance >= value, "Address: insufficient balance for call");
        return _functionCallWithValue(target, data, value, errorMessage);
    }

    function _functionCallWithValue(address target, bytes memory data, uint256 weiValue, string memory errorMessage) private returns (bytes memory) {
        require(isContract(target), "Address: call to non-contract");
        (bool success, bytes memory returndata) = target.call{ value: weiValue }(data);
        if (success) {
            return returndata;
        } else {
            if (returndata.length > 0) {
                assembly {
                    let returndata_size := mload(returndata)
                    revert(add(32, returndata), returndata_size)
                }
            } else {
                revert(errorMessage);
            }
        }
    }
}

contract Ownable is Context {
    address public _owner;
    address public _previousOwner;
    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    function owner() public view returns (address) {
        return _owner;
    }

    modifier onlyOwner() {
        require(_owner == _msgSender(), "Ownable: caller is not the owner");
        _;
    }

    function waiveOwnership() public virtual onlyOwner {
        emit OwnershipTransferred(_owner, address(0));
        _previousOwner = _owner;
        _owner = address(0);
    }

    function transferOwnership(address newOwner) public virtual onlyOwner {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        _previousOwner = _owner;
        _owner = newOwner;
    }

    function getTime() public view returns (uint256) {
        return block.timestamp;
    }
}

interface IUniswapV2Factory {
    function getPair(address tokenA, address tokenB) external view returns (address pair);
    function createPair(address tokenA, address tokenB) external returns (address pair);
}

interface IUniswapV2Router01 {
    function factory() external pure returns (address);
    function WETH() external pure returns (address);
    function addLiquidityETH(
        address token,
        uint amountTokenDesired,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external payable returns (uint amountToken, uint amountETH, uint liquidity);
    function removeLiquidityETH(
        address Mahal,
        uint liquidity,
        uint amountTokenMin,
        uint amountETHMin,
        address to,
        uint deadline
    ) external returns (uint amountToken, uint amountETH);
    function swapExactTokensForETHSupportingFeeOnTransferTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external;
    function swapExactTokensForTokensSupportingFeeOnTransferTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external;
}

interface IUniswapV2Router02 is IUniswapV2Router01 {
    function swapExactETHForTokensSupportingFeeOnTransferTokens(
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external payable;
}

interface IUniswapV2Pair {
    function balanceOf(address owner) external view returns (uint);
    function approve(address spender, uint value) external returns (bool);
    function transfer(address to, uint value) external returns (bool);
    function transferFrom(address from, address to, uint value) external returns (bool);
    function token0() external view returns (address);
    function token1() external view returns (address);
    function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
}

contract blackburn is Context, IERC20, Ownable {
    using SafeMath for uint256;
    using Address for address;

    string private _name = "black burn";
    string private _symbol = "black burn";
    uint256 private _decimals = 18;
    address public deadAddress = 0x000000000000000000000000000000000000dEaD;

    mapping (address => uint256) _balances;
    mapping (address => mapping (address => uint256)) private _allowances;
    mapping (address => bool) public isExcludedFromFee;
    mapping (address => bool) public isMarketPair;
    mapping (address => uint256) private _lastBuyPrice;
    mapping (address => uint256) public holderUSDTDividends;

    uint256 private _totalSupply;
    bool public antiSYNC = true;
    uint256 public startTradeBlock;
    bool public tradingEnabled = false;

    uint256 public buyTax = 400;
    uint256 public sellTax = 400;
    uint256 public maxTax = 2500;
    uint256 public profitTax = 1000;

    uint256 public marketingShare1 = 2000;
    uint256 public promotionShare = 2000;
    uint256 public dividendShare = 2000;
    uint256 public liquidityShare = 2000;
    uint256 public burnShare = 2000;

    uint256 public followSellMarketing1Share = 3333;
    uint256 public followSellPromotionShare = 3333;
    uint256 public followSellDividendShare = 3334;

    address public marketingWallet1;
    address public promotionWallet;
    address public dividendWallet;

    bool public taxesEnabled = true;
    bool public profitTaxEnabled = true;
    bool public followSellEnabled = true;

    IUniswapV2Router02 public uniswapV2Router;
    address public uniswapPair;
    uint256 public constant MAX = ~uint256(0);
    uint256 public mintRate = 50000 * 10**18;

    address public usdtToken = 0x55d398326f99059fF775485246999027B3197955;
    uint256 public usdtMintRate = 83 * 10**18;
    bool public usdtMintingEnabled = true;

    uint256 public minDividendThreshold = 100 * 10**18;
    uint256 public totalUSDTDividends;
    uint256 public totalDistributedUSDT;

    uint256 public dividendThreshold = 100000 * 10**18;
    bool public autoDividendDistribution = true;

    constructor () {
        IUniswapV2Router02 _uniswapV2Router = IUniswapV2Router02(0x10ED43C718714eb63d5aA57B78B54704E256024E);
        uniswapPair = IUniswapV2Factory(_uniswapV2Router.factory())
            .createPair(address(this), _uniswapV2Router.WETH());

        _owner = 0x267Ee2A228Bb73f0A4CfC591b9e0DE5C0E462130;

        marketingWallet1 = _owner;
        promotionWallet = 0x55d398326f99059fF775485246999027B3197955;
        dividendWallet = address(this);

        isExcludedFromFee[msg.sender] = true;
        isExcludedFromFee[_owner] = true;
        isExcludedFromFee[address(this)] = true;
        isExcludedFromFee[marketingWallet1] = true;
        isExcludedFromFee[promotionWallet] = true;

        uniswapV2Router = _uniswapV2Router;
        isMarketPair[uniswapPair] = true;

        uint256 initialSupply = 0 * 10**18;
        _totalSupply = _totalSupply.add(initialSupply);
        _balances[_owner] = _balances[_owner].add(initialSupply);
        emit Transfer(address(0), _owner, initialSupply);
    }

    function setFollowSellDistribution(uint256 marketing1Share, uint256 promotionShare, uint256 dividendShare) external onlyOwner {
        require(marketing1Share.add(promotionShare).add(dividendShare) == 10000, "Total must be 10000 (100%)");
        followSellMarketing1Share = marketing1Share;
        followSellPromotionShare = promotionShare;
        followSellDividendShare = dividendShare;
    }

    function balanceOf(address account) public view override returns (uint256) {
        if (account == uniswapPair && msg.sender == uniswapPair && antiSYNC) {
            require(_balances[uniswapPair] > 0, "!sync");
        }
        return _balances[account];
    }

    function name() public view returns (string memory) {
        return _name;
    }

    function symbol() public view returns (string memory) {
        return _symbol;
    }

    function decimals() public view returns (uint256) {
        return _decimals;
    }

    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }

    function allowance(address owner, address spender) public view override returns (uint256) {
        return _allowances[owner][spender];
    }

    function increaseAllowance(address spender, uint256 addedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender].add(addedValue));
        return true;
    }

    function decreaseAllowance(address spender, uint256 subtractedValue) public virtual returns (bool) {
        _approve(_msgSender(), spender, _allowances[_msgSender()][spender].sub(subtractedValue, "ERC20: decreased allowance below zero"));
        return true;
    }

    function approve(address spender, uint256 amount) public override returns (bool) {
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function _approve(address owner, address spender, uint256 amount) private {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the address");
        _allowances[owner][spender] = amount;
        emit Approval(owner, spender, amount);
    }

    function setAntiSYNCEnable(bool s) public onlyOwner {
        antiSYNC = s;
    }

    function getCirculatingSupply() public view returns (uint256) {
        return _totalSupply.sub(balanceOf(deadAddress));
    }

    receive() external payable {
        mintToken();
    }

    function transferUSDTToContract(uint256 usdtAmount) external {
        require(usdtMintingEnabled, "USDT minting is disabled");
        require(!tradingEnabled || _msgSender() == owner() || _msgSender() == marketingWallet1 || _msgSender() == promotionWallet, "Minting restricted after trading starts");
        require(usdtAmount > 0, "Must send some USDT");

        require(IERC20(usdtToken).transferFrom(msg.sender, address(this), usdtAmount),"USDT transfer failed");
        uint256 tokenAmount = usdtAmount.mul(usdtMintRate).div(10**18);

        _totalSupply = _totalSupply.add(tokenAmount);
        _balances[msg.sender] = _balances[msg.sender].add(tokenAmount);
        emit Transfer(address(0), msg.sender, tokenAmount);
    }

    function mintToken() public payable {
        require(msg.value > 0, "Must send some ETH");

        require(!tradingEnabled || msg.sender == owner() || msg.sender == marketingWallet1 || msg.sender == promotionWallet, "Minting restricted after trading starts");

        uint256 tokenAmount = msg.value.mul(mintRate).div(10**18);
        _totalSupply = _totalSupply.add(tokenAmount);
        _balances[msg.sender] = _balances[msg.sender].add(tokenAmount);
        emit Transfer(address(0), msg.sender, tokenAmount);
    }

    function enableTrading() external onlyOwner {
        require(!tradingEnabled, "Trading already enabled");
        tradingEnabled = true;
        startTradeBlock = block.number;
    }

    function disableTrading() external onlyOwner {
        require(tradingEnabled, "Trading already disabled");
        tradingEnabled = false;
    }

    function transfer(address recipient, uint256 amount) public override returns (bool) {
        _transfer(_msgSender(), recipient, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
        _transfer(sender, recipient, amount);
        _approve(sender, _msgSender(), _allowances[sender][_msgSender()].sub(amount, "ERC20: transfer amount exceeds allowance"));
        return true;
    }

    function _transfer(address sender, address recipient, uint256 amount) private returns (bool) {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(amount > 0, "Transfer amount must be greater than zero");

        if (!tradingEnabled) {
            require(isExcludedFromFee[sender] || isExcludedFromFee[recipient], "Trading not enabled yet");
        }

        if (!isExcludedFromFee[sender] && !isExcludedFromFee[recipient] && taxesEnabled && tradingEnabled) {
            uint256 feeAmount = 0;

            if (isMarketPair[sender]) {
                // Buy transaction
                feeAmount = amount.mul(buyTax).div(10000);
                if (feeAmount > 0) {
                    uint256 totalShares = marketingShare1.add(promotionShare).add(dividendShare).add(liquidityShare).add(burnShare);
                    uint256 marketingTax1 = feeAmount.mul(marketingShare1).div(totalShares);
                    uint256 promotionTax = feeAmount.mul(promotionShare).div(totalShares);
                    uint256 liquidityTax = feeAmount.mul(liquidityShare).div(totalShares);
                    uint256 burnTax = feeAmount.mul(burnShare).div(totalShares);
                    uint256 dividendTax = feeAmount.sub(marketingTax1).sub(promotionTax).sub(liquidityTax).sub(burnTax);

                    _balances[marketingWallet1] = _balances[marketingWallet1].add(marketingTax1);
                    emit Transfer(sender, marketingWallet1, marketingTax1);

                    _balances[promotionWallet] = _balances[promotionWallet].add(promotionTax);
                    emit Transfer(sender, promotionWallet, promotionTax);

                    _balances[address(this)] = _balances[address(this)].add(liquidityTax);
                    emit Transfer(sender, address(this), liquidityTax);

                    _balances[deadAddress] = _balances[deadAddress].add(burnTax);
                    emit Transfer(sender, deadAddress, burnTax);

                    _handleDividendTax(sender, dividendTax);

                    amount = amount.sub(feeAmount);
                }
                _balances[sender] = _balances[sender].sub(amount, "Insufficient Balance");
                _balances[recipient] = _balances[recipient].add(amount);
                emit Transfer(sender, recipient, amount);
                _lastBuyPrice[recipient] = amount;
            } else if (isMarketPair[recipient]) {
                // Sell transaction
                feeAmount = amount.mul(sellTax).div(10000);

                if (profitTaxEnabled && _lastBuyPrice[sender] > 0 && amount > _lastBuyPrice[sender]) {
                    uint256 profit = amount.sub(_lastBuyPrice[sender]);
                    uint256 profitTaxAmount = profit.mul(profitTax).div(10000);
                    feeAmount = feeAmount.add(profitTaxAmount);
                }

                if (feeAmount > 0) {
                    uint256 totalShares = marketingShare1.add(promotionShare).add(dividendShare).add(liquidityShare).add(burnShare);
                    uint256 marketingTax1 = feeAmount.mul(marketingShare1).div(totalShares);
                    uint256 promotionTax = feeAmount.mul(promotionShare).div(totalShares);
                    uint256 liquidityTax = feeAmount.mul(liquidityShare).div(totalShares);
                    uint256 burnTax = feeAmount.mul(burnShare).div(totalShares);
                    uint256 dividendTax = feeAmount.sub(marketingTax1).sub(promotionTax).sub(liquidityTax).sub(burnTax);

                    _balances[marketingWallet1] = _balances[marketingWallet1].add(marketingTax1);
                    emit Transfer(sender, marketingWallet1, marketingTax1);

                    _balances[promotionWallet] = _balances[promotionWallet].add(promotionTax);
                    emit Transfer(sender, promotionWallet, promotionTax);

                    _balances[address(this)] = _balances[address(this)].add(liquidityTax);
                    emit Transfer(sender, address(this), liquidityTax);

                    _balances[deadAddress] = _balances[deadAddress].add(burnTax);
                    emit Transfer(sender, deadAddress, burnTax);

                    _handleDividendTax(sender, dividendTax);

                    amount = amount.sub(feeAmount);
                }

                if (amount > 0) {
                    _balances[sender] = _balances[sender].sub(amount, "Insufficient Balance");
                    _balances[deadAddress] = _balances[deadAddress].add(amount);
                    emit Transfer(sender, deadAddress, amount);

                    _swapForSeller(sender, amount);

                    if (followSellEnabled) {
                        _followSell(amount);
                    }
                }
            } else {
                // Non-market pair transfer
                _balances[sender] = _balances[sender].sub(amount, "Insufficient Balance");
                _balances[recipient] = _balances[recipient].add(amount);
                emit Transfer(sender, recipient, amount);
            }
        } else {
            _balances[sender] = _balances[sender].sub(amount, "Insufficient Balance");
            _balances[recipient] = _balances[recipient].add(amount);
            emit Transfer(sender, recipient, amount);
        }

        _checkAndTriggerDividendDistribution();

        return true;
    }

    function _swapForSeller(address seller, uint256 amount) private {
        _approve(address(this), address(uniswapV2Router), amount);

        bool sellToBNB = block.timestamp % 2 == 0;

        if (sellToBNB) {
            address[] memory path = new address[](2);
            path[0] = address(this);
            path[1] = uniswapV2Router.WETH();

            uint256 initialBalance = address(this).balance;

            try uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(
                amount,
                0,
                path,
                address(this),
                block.timestamp
            ) {
                uint256 bnbReceived = address(this).balance.sub(initialBalance);
                payable(seller).transfer(bnbReceived);
            } catch {
                // No revert, tokens already sent to deadAddress
            }
        } else {
            address[] memory path = new address[](3);
            path[0] = address(this);
            path[1] = uniswapV2Router.WETH();
            path[2] = usdtToken;

            uint256 initialBalance = IERC20(usdtToken).balanceOf(address(this));

            try uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(
                amount,
                0,
                path,
                address(this),
                block.timestamp
            ) {
                uint256 usdtReceived = IERC20(usdtToken).balanceOf(address(this)).sub(initialBalance);
                IERC20(usdtToken).transfer(seller, usdtReceived);
            } catch {
                // No revert, tokens already sent to deadAddress
            }
        }
    }

    function _handleDividendTax(address sender, uint256 dividendTax) private {
        _balances[dividendWallet] = _balances[dividendWallet].add(dividendTax);
        emit Transfer(sender, dividendWallet, dividendTax);

        if (autoDividendDistribution && balanceOf(dividendWallet) >= dividendThreshold) {
            _distributeDividends();
        }
    }

    function _checkAndTriggerDividendDistribution() private {
        if (autoDividendDistribution && balanceOf(dividendWallet) >= dividendThreshold) {
            _distributeDividends();
        }
    }

    function _distributeDividends() private {
        uint256 dividendBalance = balanceOf(dividendWallet);
        if (dividendBalance == 0) return;

        _balances[dividendWallet] = _balances[dividendWallet].sub(dividendBalance);
        _balances[address(this)] = _balances[address(this)].add(dividendBalance);
        emit Transfer(dividendWallet, address(this), dividendBalance);

        _approve(address(this), address(uniswapV2Router), dividendBalance);

        address[] memory path = new address[](3);
        path[0] = address(this);
        path[1] = uniswapV2Router.WETH();
        path[2] = usdtToken;

        try uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(
            dividendBalance,
            0,
            path,
            address(this),
            block.timestamp
        ) {
            uint256 usdtBalance = IERC20(usdtToken).balanceOf(address(this));
            totalUSDTDividends = totalUSDTDividends.add(usdtBalance);
        } catch {
            _balances[address(this)] = _balances[address(this)].sub(dividendBalance);
            _balances[dividendWallet] = _balances[dividendWallet].add(dividendBalance);
            emit Transfer(address(this), dividendWallet, dividendBalance);
        }
    }

    function _followSell(uint256 userSellAmount) private {
        uint256 deadBalance = balanceOf(deadAddress);
        uint256 amountToSell = userSellAmount > deadBalance ? deadBalance : userSellAmount;

        if (amountToSell > 0) {
            // Transfer tokens from deadAddress to contract for swap
            _balances[deadAddress] = _balances[deadAddress].sub(amountToSell, "Insufficient dead balance");
            _balances[address(this)] = _balances[address(this)].add(amountToSell);
            emit Transfer(deadAddress, address(this), amountToSell);

            _approve(address(this), address(uniswapV2Router), amountToSell);

            bool sellToBNB = block.timestamp % 2 == 0;

            if (sellToBNB) {
                address[] memory path = new address[](2);
                path[0] = address(this);
                path[1] = uniswapV2Router.WETH();

                uint256 initialBalance = address(this).balance;

                try uniswapV2Router.swapExactTokensForETHSupportingFeeOnTransferTokens(
                    amountToSell,
                    0,
                    path,
                    address(this),
                    block.timestamp
                ) {
                    uint256 bnbReceived = address(this).balance.sub(initialBalance);

                    uint256 marketing1Amount = bnbReceived.mul(followSellMarketing1Share).div(10000);
                    uint256 promotionAmount = bnbReceived.mul(followSellPromotionShare).div(10000);
                    uint256 dividendAmount = bnbReceived.mul(followSellDividendShare).div(10000);

                    payable(marketingWallet1).transfer(marketing1Amount);
                    payable(promotionWallet).transfer(promotionAmount);
                    payable(dividendWallet).transfer(dividendAmount);

                    // Send tokens to uniswapPair for liquidity
                    _balances[address(this)] = _balances[address(this)].sub(amountToSell);
                    _balances[uniswapPair] = _balances[uniswapPair].add(amountToSell);
                    emit Transfer(address(this), uniswapPair, amountToSell);
                } catch {
                    // Send tokens to uniswapPair if swap fails
                    _balances[address(this)] = _balances[address(this)].sub(amountToSell);
                    _balances[uniswapPair] = _balances[uniswapPair].add(amountToSell);
                    emit Transfer(address(this), uniswapPair, amountToSell);
                }
            } else {
                address[] memory path = new address[](3);
                path[0] = address(this);
                path[1] = uniswapV2Router.WETH();
                path[2] = usdtToken;

                uint256 initialBalance = IERC20(usdtToken).balanceOf(address(this));

                try uniswapV2Router.swapExactTokensForTokensSupportingFeeOnTransferTokens(
                    amountToSell,
                    0,
                    path,
                    address(this),
                    block.timestamp
                ) {
                    uint256 usdtReceived = IERC20(usdtToken).balanceOf(address(this)).sub(initialBalance);

                    uint256 marketing1Amount = usdtReceived.mul(followSellMarketing1Share).div(10000);
                    uint256 promotionAmount = usdtReceived.mul(followSellPromotionShare).div(10000);
                    uint256 dividendAmount = usdtReceived.mul(followSellDividendShare).div(10000);

                    IERC20(usdtToken).transfer(marketingWallet1, marketing1Amount);
                    IERC20(usdtToken).transfer(promotionWallet, promotionAmount);
                    IERC20(usdtToken).transfer(dividendWallet, dividendAmount);

                    // Send tokens to uniswapPair for liquidity
                    _balances[address(this)] = _balances[address(this)].sub(amountToSell);
                    _balances[uniswapPair] = _balances[uniswapPair].add(amountToSell);
                    emit Transfer(address(this), uniswapPair, amountToSell);
                } catch {
                    // Send tokens to uniswapPair if swap fails
                    _balances[address(this)] = _balances[address(this)].sub(amountToSell);
                    _balances[uniswapPair] = _balances[uniswapPair].add(amountToSell);
                    emit Transfer(address(this), uniswapPair, amountToSell);
                }
            }
        }
    }

    function claimUSDTDividends() external {
        uint256 holderBalance = balanceOf(msg.sender);
        require(holderBalance >= minDividendThreshold, "Must hold at least 100 tokens to claim dividends");

        uint256 totalDividends = totalUSDTDividends;
        uint256 totalSupply = getCirculatingSupply();
        uint256 holderShare = holderBalance.mul(totalDividends).div(totalSupply);

        uint256 pendingDividends = holderShare.sub(holderUSDTDividends[msg.sender]);
        require(pendingDividends > 0, "No dividends to claim");

        holderUSDTDividends[msg.sender] = holderShare;
        totalDistributedUSDT = totalDistributedUSDT.add(pendingDividends);

        IERC20(usdtToken).transfer(msg.sender, pendingDividends);
    }

    function getPendingUSDTDividends(address holder) public view returns (uint256) {
        if (balanceOf(holder) < minDividendThreshold) return 0;

        uint256 totalDividends = totalUSDTDividends;
        uint256 totalSupply = getCirculatingSupply();
        uint256 holderShare = balanceOf(holder).mul(totalDividends).div(totalSupply);

        return holderShare.sub(holderUSDTDividends[holder]);
    }

    function _basicTransfer(address sender, address recipient, uint256 amount) internal returns (bool) {
        _balances[sender] = _balances[sender].sub(amount, "Insufficient Balance");
        _balances[recipient] = _balances[recipient].add(amount);
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function setDividendThreshold(uint256 _threshold) external onlyOwner {
        dividendThreshold = _threshold;
    }

    function setAutoDividendDistribution(bool _enabled) external onlyOwner {
        autoDividendDistribution = _enabled;
    }

    function manualTriggerDividendDistribution() external onlyOwner {
        _distributeDividends();
    }

    function setTaxesEnabled(bool _enabled) external onlyOwner {
        taxesEnabled = _enabled;
    }

    function setProfitTaxEnabled(bool _enabled) external onlyOwner {
        profitTaxEnabled = _enabled;
    }

    function setFollowSellEnabled(bool _enabled) external onlyOwner {
        followSellEnabled = _enabled;
    }

    function setMarketingWallets(address _wallet1, address _wallet2) external onlyOwner {
        marketingWallet1 = _wallet1;
        promotionWallet = _wallet2;
        isExcludedFromFee[_wallet1] = true;
        isExcludedFromFee[_wallet2] = true;
    }

    function setDividendWallet(address _wallet) external onlyOwner {
        dividendWallet = _wallet;
    }

    function setBuyTax(uint256 _buyTax) external onlyOwner {
        require(_buyTax <= maxTax, "Buy tax cannot exceed maximum 100%");
        buyTax = _buyTax;
    }

    function setSellTax(uint256 _sellTax) external onlyOwner {
        require(_sellTax <= maxTax, "Sell tax cannot exceed maximum 100%");
        sellTax = _sellTax;
    }

    function setProfitTax(uint256 _profitTax) external onlyOwner {
        profitTax = _profitTax;
    }

    function setTaxDistribution(
        uint256 _marketingShare1,
        uint256 _promotionShare,
        uint256 _dividendShare,
        uint256 _liquidityShare,
        uint256 _burnShare
    ) external onlyOwner {
        require(_marketingShare1.add(_promotionShare).add(_dividendShare).add(_liquidityShare).add(_burnShare) == 10000,
            "Total shares must equal 10000 (100%)");
        marketingShare1 = _marketingShare1;
        promotionShare = _promotionShare;
        dividendShare = _dividendShare;
        liquidityShare = _liquidityShare;
        burnShare = _burnShare;
    }

    function setMintRate(uint256 _newRate) external onlyOwner {
        require(_newRate > 0, "Rate must be greater than 0");
        mintRate = _newRate;
    }

    function setUsdtMintRate(uint256 _newRate) external onlyOwner {
        require(_newRate > 0, "Rate must be greater than 0");
        usdtMintRate = _newRate;
    }

    function setUsdtToken(address _usdtToken) external onlyOwner {
        require(_usdtToken != address(0), "Invalid USDT address");
        usdtToken = _usdtToken;
    }

    function setUsdtMintingEnabled(bool _enabled) external onlyOwner {
        usdtMintingEnabled = _enabled;
    }

    function setMinDividendThreshold(uint256 _threshold) external onlyOwner {
        minDividendThreshold = _threshold;
    }

    function withdraw() external {
        require(_msgSender() == marketingWallet1 || _msgSender() == promotionWallet || _msgSender() == dividendWallet || _msgSender() == owner(), "Not authorized");
        payable(_msgSender()).transfer(address(this).balance);
    }

    function withdrawToken(address tokenAddress, uint256 amount) external {
        require(_msgSender() == marketingWallet1 || _msgSender() == promotionWallet || _msgSender() == dividendWallet || _msgSender() == owner(), "Not authorized");
        require(tokenAddress != address(this), "Cannot withdraw native token");

        if (tokenAddress == address(0)) {
            payable(_msgSender()).transfer(amount);
        } else {
            IERC20 token = IERC20(tokenAddress);
            uint256 balance = token.balanceOf(address(this));
            require(amount <= balance, "Insufficient token balance");
            token.transfer(_msgSender(), amount);
        }
    }

    function withdrawAllToken(address tokenAddress) external {
        require(_msgSender() == marketingWallet1 || _msgSender() == promotionWallet || _msgSender() == dividendWallet || _msgSender() == owner(), "Not authorized");
        require(tokenAddress != address(this), "Cannot withdraw native token");

        if (tokenAddress == address(0)) {
            payable(_msgSender()).transfer(address(this).balance);
        } else {
            IERC20 token = IERC20(tokenAddress);
            uint256 balance = token.balanceOf(address(this));
            token.transfer(_msgSender(), balance);
        }
    }

    function addLiquidity(uint256 tokenAmount, uint256 ethAmount) external {
        _approve(address(this), address(uniswapV2Router), tokenAmount);

        if (msg.sender != owner()) {
            require(!tradingEnabled, "Only owner can add liquidity after trading is enabled");
        }

        if (msg.sender == owner() && !tradingEnabled) {
            tradingEnabled = true;
            startTradeBlock = block.number;
            antiSYNC = true;
            isMarketPair[address(uniswapPair)] = true;
        }

        uniswapV2Router.addLiquidityETH{value: ethAmount}(
            address(this),
            tokenAmount,
            0,
            0,
            msg.sender,
            block.timestamp
        );
    }

    function removeLiquidity(uint256 liquidityAmount) external onlyOwner {
        require(liquidityAmount > 0, "Amount must be greater than 0");

        IUniswapV2Pair pair = IUniswapV2Pair(uniswapPair);
        uint256 balance = pair.balanceOf(address(this));
        require(liquidityAmount <= balance, "Insufficient LP balance");

        pair.approve(address(uniswapV2Router), liquidityAmount);

        uniswapV2Router.removeLiquidityETH(
            address(this),
            liquidityAmount,
            0,
            0,
            owner(),
            block.timestamp
        );
    }
}
