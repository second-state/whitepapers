---
description: Next generation Ethereum smart contracts at your finger tip
---

# Run an ewasm smart contract

One of the key features of the Second State DevChain is its support for the next-gen Ethereum virtual machine, Ewasm. Smart contracts compiled to Ewasm can be deployed on the DevChain. 

{% hint style="warning" %}
Before you start, please make sure that you have a Second State DevChain up and running. If not, follow the [Getting Started tutorial](https://docs.secondstate.io/devchain/getting-started). It takes about 10 minutes.
{% endhint %}

You can use the [SOLL compiler](https://github.com/second-state/soll) to compile Solidity or YUL source code to Ewasm bytecode. Follow the[ instructions here](https://github.com/second-state/SOLL/blob/master/README.md). The Solidity source code the smart contract is as follows. It is a simplified ERC20 contract.

```text
pragma solidity ^0.5.0;
contract Token {
	uint256 private totalSupply;
	string public name;
	string public symbol;
	mapping(address => uint256) public balances;

	event Transfer(address indexed _from, address indexed _to, uint256 _value);

	// Safemath
	function add(uint256 a, uint256 b) internal pure returns (uint256) {
		uint256 c = a + b;
		require(c >= a, "SafeMath: addition overflow");

		return c;
	}

	function sub(uint256 a, uint256 b) internal pure returns (uint256) {
		require(b <= a, "SafeMath: subtraction overflow");
		uint256 c = a - b;

		return c;
	}

	constructor() public {
		totalSupply = 100000000;
		name = "ERC20TokenDemo";
		symbol = "ETD";
		balances[msg.sender] = totalSupply;
	}

	function balanceOf(address account) view public returns (uint256) {
		return balances[account];
	}

	function transfer(address to, uint256 amount) public returns (bool) {
		balances[msg.sender] = sub(balances[msg.sender], amount);
		balances[to] = add(balances[to], amount);
		emit Transfer(msg.sender, to, amount);
		return true;
	}

	function () external payable {}
}
```

The compiled ewasm bytecode is converted to text using the `xxd` tool.

```text
$ xxd -p contract.wasm | tr -d $'\n'
0061736d0100000001270760027f7f0060000060017f0060037f7f7f0060057f7e7e7e7e0060047e7f7f7f017f6000017e02c7010908657468657265756d0c67657443616c6c56616c7565000208657468657265756d0c73746f7261676553746f7265000008657468657265756d0967657443616c6c6572000208657468657265756d0a6765744761734c656674000608657468657265756d0a63616c6c537461746963000508657468657265756d0e72657475726e44617461436f7079000308657468657265756d0b73746f726167654c6f6164000008657468657265756d06726576657274000008657468657265756d0666696e697368000003040304010105030100020608017f0141e0ad040b071102066d656d6f72790200046d61696e000b0abf0d038d0300200020044228884280fe03832004421888428080fc0783200442088842808080f80f8320044208864280808080f01f832004421886428080808080e03f83200442288642808080808080c0ff00832004423886200442388884848484848484370300200041186a20014228884280fe03832001421888428080fc0783200142088842808080f80f8320014208864280808080f01f832001421886428080808080e03f83200142288642808080808080c0ff00832001423886848484848484200142388884370300200020024228884280fe03832002421888428080fc0783200242088842808080f80f8320024208864280808080f01f832002421886428080808080e03f83200242288642808080808080c0ff00832002423886848484848484200242388884370310200020034228884280fe03832003421888428080fc0783200342088842808080f80f8320034208864280808080f01f832003421886428080808080e03f83200342288642808080808080c0ff008320034238868484848484842003423888843703080ba00a02067f087e23004190026b220024002000220141c8016a100020012903c801200141d0016a29030084500440200041606a220322022400200241606a220422052400200241786a4280808080d0a0fdf000370300200241706a4200370300200241686a420037030020044200370300200041786a4200370300200041706a4200370300200041686a4200370300200342003703002003200410012005220041606a2202220324002000416c6a41edde013b0100200041686a220441e5dc91aa06360200200242c5a48d928386d5b7eb00370300200041786a220220022903004280808080808080801c84220637030020042903002107200041706a29030021082003220041606a220322022400200241606a220422052400200241786a2006370300200241706a2008370300200241686a2007370300200442c5a48d928386d5b7eb00370300200041786a42808080808080808001370300200041706a4200370300200041686a4200370300200342003703002003200410012005220041606a220222032400200041626a41c4003a0000200241c5a8013b0100200041786a220420042903004280808080808080800684220637030020022903002107200041686a2903002108200041706a29030021092003220041606a220322022400200241606a220422052400200241786a2006370300200241706a2009370300200241686a200837030020042007370300200041786a42808080808080808002370300200041706a4200370300200041686a4200370300200342003703002003200410012005220041606a22022203240020021002200141a8016a4200200229030022064220862006422088200041686a290300220642208684200041706a350200422086200642208884100920014188016a20012903a801200141b0016a290300200141b8016a3502004200100920014190016a290300210620014198016a2903002107200141a0016a290300210820012903880121092003220041406a220222032400200041786a42808080808080808003370300200041706a4200370300200041686a4200370300200041606a4200370300200041586a2008370300200041506a2007370300200041486a2006370300200220093703002001420037038002200142003703f801200142808080103e0288021003200141f8016a200241c00010041a200141d8016a410041201005200141e8006a20012903d801200141e0016a290300200141e8016a290300200141f0016a290300100920014180016a2903002106200141f8006a2903002107200141f0006a2903002108200129036821092003220041606a220322022400200241606a220422052400200041786a4200370300200041706a4200370300200041686a420037030020034200370300200320041006200141c8006a2004290300200241686a290300200241706a290300200241786a2903001009200141286a20092008200720061009200141306a2903002106200141386a2903002107200141406b290300210820012903282109200141086a2001290348200141d0006a290300200141d8006a290300200141e0006a2903001009200141106a290300210a200141186a290300210b200141206a290300210c2001290308210d2005220041606a220322022400200241606a22042400200241786a200c370300200241706a200b370300200241686a200a3703002004200d370300200041786a2008370300200041706a2007370300200041686a20063703002003200937030020032004100120014190026a24000f0b41800841171007000b0c00100a41970841c42510080b0be32501004180080bdb2546756e6374696f6e206973206e6f742070617961626c650061736d0100000001460b60027f7f0060017f0060037f7f7f0060000060077f7f7f7f7f7f7f0060047f7e7e7e0060057f7e7e7e7e0060077e7e7e7e7e7e7e006000017f60047e7f7f7f017f6000017e0289020c08657468657265756d0c67657443616c6c56616c7565000108657468657265756d0a6765744761734c656674000a08657468657265756d0a63616c6c537461746963000908657468657265756d0e72657475726e44617461436f7079000208657468657265756d0b73746f726167654c6f6164000008657468657265756d06726576657274000008657468657265756d0967657443616c6c6572000108657468657265756d0c73746f7261676553746f7265000008657468657265756d036c6f67000408657468657265756d0f67657443616c6c4461746153697a65000808657468657265756d0c63616c6c44617461436f7079000208657468657265756d0666696e69736800000305040605070305030100020608017f0141f088040b071102066d656d6f72790200046d61696e000f0ac321048d0300200020044228884280fe03832004421888428080fc0783200442088842808080f80f8320044208864280808080f01f832004421886428080808080e03f83200442288642808080808080c0ff00832004423886200442388884848484848484370300200041186a20014228884280fe03832001421888428080fc0783200142088842808080f80f8320014208864280808080f01f832001421886428080808080e03f83200142288642808080808080c0ff00832001423886848484848484200142388884370300200020024228884280fe03832002421888428080fc0783200242088842808080f80f8320024208864280808080f01f832002421886428080808080e03f83200242288642808080808080c0ff00832002423886848484848484200242388884370310200020034228884280fe03832003421888428080fc0783200342088842808080f80f8320034208864280808080f01f832003421886428080808080e03f83200342288642808080808080c0ff008320034238868484848484842003423888843703080baf0402057f017e230041d0016b22052400200522044188016a100020042903880120044190016a29030084500440200441e8006a20012002200342ffffffff0f834200100c200441f0006a2903002101200441f8006a290300210220044180016a290300210320042903682109200541406a220622072400200541786a42808080808080808003370300200541706a4200370300200541686a4200370300200541606a4200370300200541586a2003370300200541506a2002370300200541486a200137030020062009370300200442003703c001200442003703b801200442808080103e02c8011001200441b8016a200641c00010021a20044198016a410041201003200441c8006a200429039801200441a0016a290300200441a8016a290300200441b0016a290300100c200441286a2004290348200441d0006a290300200441d8006a290300200441e0006a290300100c200441306a2903002101200441386a2903002102200441406b2903002103200429032821092007220541606a220722062400200641606a22082400200541786a2003370300200541706a2002370300200541686a200137030020072009370300200720081004200441086a2008290300200641686a290300200641706a290300200641786a290300100c200441106a2903002101200441186a290300210220042903082103200041186a200441206a290300370300200020023703102000200137030820002003370300200441d0016a24000f0b41d90841171005000bec1502077f0b7e230041d0056b22082400200822074188056a1000024002400240024020072903880520074190056a29030084500440200841606a2209220b240020091006200741e8046a42002009290300220e422086200e422088200841686a290300220e42208684200841706a350200422086200e42208884100c200741c8046a20072903e804200741f0046a290300200741f8046a3502004200100c200741d0046a290300210e200741d8046a290300210f200741e0046a290300211020072903c8042112200b220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2010370300200841506a200f370300200841486a200e37030020092012370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a410041201003200741a8046a200729039805200741a0056a220b290300200741a8056a220c290300200741b0056a220d290300100c200741c0046a2903002111200741b8046a2903002113200741b0046a290300211420072903a8042115200a220841606a2209220a24002009100620074188046a42002009290300220e422086200e422088200841686a290300220e42208684200841706a350200422086200e42208884100c200741e8036a20072903880420074190046a29030020074198046a3502004200100c200741f0036a290300210e200741f8036a290300210f20074180046a290300211020072903e8032112200a220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2010370300200841506a200f370300200841486a200e37030020092012370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a410041201003200741c8036a200729039805200b290300200c290300200d290300100c200741a8036a20072903c803200741d0036a290300200741d8036a290300200741e0036a290300100c200741b0036a290300210e200741b8036a290300210f200741c0036a290300211020072903a8032112200a220841606a220a22092400200941606a220c220d2400200841786a2010370300200841706a200f370300200841686a200e370300200a2012370300200a200c100420074188036a200c290300200941686a290300200941706a290300200941786a290300100c200741a0036a290300210e20074198036a290300210f20074190036a2903002110200729038803211220074198056a1000200729039805200b290300844200520d0120122003542208201020045420042010511b2209200f200554220b200e2006542006200e511b2005200f852006200e8584501b0d02200741e8026a2015201420132011100c200741f0026a2903002111200741f8026a290300211320074180036a290300211420072903e8022115200741c8026a201220037d201020047d2008ad7d200f20057d220f2009ad22107d200e20067d200bad7d200f201054ad7d100c200741d0026a290300210e200741d8026a290300210f200741e0026a290300211020072903c8022112200d220841606a220b22092400200941606a220a220c2400200941786a2010370300200941706a200f370300200941686a200e370300200a2012370300200841786a2014370300200841706a2013370300200841686a2011370300200b2015370300200b200a1007200741a8026a20002001200242ffffffff0f834200100c200741b0026a2903002110200741b8026a2903002112200741c0026a290300210020072903a8022101200c220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2000370300200841506a2012370300200841486a201037030020092001370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a41004120100320074188026a200729039805200741a0056a220b290300200741a8056a220c290300200741b0056a220d290300100c200741a0026a290300211420074198026a290300211520074190026a29030021162007290388022117200a220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2000370300200841506a2012370300200841486a201037030020092001370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a410041201003200741e8016a200729039805200b290300200c290300200d290300100c200741c8016a20072903e801200741f0016a290300200741f8016a29030020074180026a290300100c200741d0016a290300210e200741d8016a290300210f200741e0016a290300210220072903c8012111200a220841606a220a22092400200941606a220c220d2400200841786a2002370300200841706a200f370300200841686a200e370300200a2011370300200a200c1004200741a8016a200c290300200941686a290300200941706a290300200941786a290300100c200741c0016a290300210e200741b8016a290300210f200741b0016a290300210220072903a801211120074198056a1000200729039805200b290300844200520d03200320117c221820115422082008ad200220047c7c221320025420022013511b22082005200f7c22112008ad7c2202200f542002201154ad2011200f54ad2006200e7c7c7c2211200e54200e2011511b2002200f85200e20118584501b0d0420074188016a2017201620152014100c20074190016a290300210e20074198016a290300210f200741a0016a29030021142007290388012115200741e8006a2018201320022011100c200741f0006a2903002102200741f8006a290300211120074180016a290300211320072903682116200d220841606a220b22092400200941606a220a220c2400200941786a2013370300200941706a2011370300200941686a2002370300200a2016370300200841786a2014370300200841706a200f370300200841686a200e370300200b2015370300200b200a1007200c220841606a2209220b240020091006200741c8006a42002009290300220e422086200e422088200841686a290300220e42208684200841706a350200422086200e42208884100c200741d0006a290300210e200741d8006a350200210f20072903482102200b220841606a2209220b2400200741286a2002200e200f4200100c200741306a290300210e200741386a290300210f20072903282102200841786a200741406b290300370300200841706a200f370300200841686a200e37030020092002370300200b220841606a220b220a2400200841786a2000370300200841706a2012370300200841686a2010370300200b2001370300200a220841606a220a2400200741086a2003200420052006100c200741106a2903002106200741186a290300210420072903082105200841786a200741206a290300370300200841706a2004370300200841686a2006370300200a2005370300200a412041034180082009200b41001008200741d0056a24000f0b41d90841171005000b41d90841171005000b41bb08411e1005000b41d90841171005000b41a008411b1005000b920402047f047e230041a0016b2200210120002400024002400240100941034d0d00200041706a220022022400200041004104100a2000280200220041a98bf0dc7b460d01200041f0c08a8c03470d002002220041606a220222032400200241044120100a200141406b2002290300200041686a290300200041706a290300200041786a290300100c200141206a2001290340200141c8006a290300200141d0006a350200100d20012001290320200141286a290300200141306a290300200141386a290300100c200141086a2903002104200141106a2903002105200141186a2903002106200129030021072003220041606a22022400200041786a2006370300200041706a2005370300200041686a20043703002002200737030020024120100b0c020b41004100100b0c010b2002220041406a2202220324002002410441c000100a20014180016a2002290300200041486a290300200041506a290300200041586a290300100c200141e0006a200041606a290300200041686a290300200041706a290300200041786a290300100c20012903800120014188016a29030020014190016a3502002001290360200141e8006a290300200141f0006a290300200141f8006a290300100e2003220041606a22022400200041786a42808080808080808001370300200041706a4200370300200041686a42003703002002420037030020024120100b0b200141a0016a24000b0b7701004180080b70ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef536166654d6174683a206164646974696f6e206f766572666c6f77536166654d6174683a207375627472616374696f6e206f766572666c6f7746756e6374696f6e206973206e6f742070617961626c65
```

Now, let's prefix the ewasm code with `0x` and deploy it to the blockchain from the default coinbase address. Recall that the default passphrase for the coinbase is `1234`. All you need is an address that has some CMTs in it.

```text
> personal.unlockAccount("0x7eff122b94897ea5b0e2a9abf47b86337fafebdc")
Passphrase:
> cmt.sendTransaction({
    from: "0x7eff122b94897ea5b0e2a9abf47b86337fafebdc",
    gas: 5000000,
    data: "0x0061736d0100000001270760027f7f0060000060017f0060037f7f7f0060057f7e7e7e7e0060047e7f7f7f017f6000017e02c7010908657468657265756d0c67657443616c6c56616c7565000208657468657265756d0c73746f7261676553746f7265000008657468657265756d0967657443616c6c6572000208657468657265756d0a6765744761734c656674000608657468657265756d0a63616c6c537461746963000508657468657265756d0e72657475726e44617461436f7079000308657468657265756d0b73746f726167654c6f6164000008657468657265756d06726576657274000008657468657265756d0666696e697368000003040304010105030100020608017f0141e0ad040b071102066d656d6f72790200046d61696e000b0abf0d038d0300200020044228884280fe03832004421888428080fc0783200442088842808080f80f8320044208864280808080f01f832004421886428080808080e03f83200442288642808080808080c0ff00832004423886200442388884848484848484370300200041186a20014228884280fe03832001421888428080fc0783200142088842808080f80f8320014208864280808080f01f832001421886428080808080e03f83200142288642808080808080c0ff00832001423886848484848484200142388884370300200020024228884280fe03832002421888428080fc0783200242088842808080f80f8320024208864280808080f01f832002421886428080808080e03f83200242288642808080808080c0ff00832002423886848484848484200242388884370310200020034228884280fe03832003421888428080fc0783200342088842808080f80f8320034208864280808080f01f832003421886428080808080e03f83200342288642808080808080c0ff008320034238868484848484842003423888843703080ba00a02067f087e23004190026b220024002000220141c8016a100020012903c801200141d0016a29030084500440200041606a220322022400200241606a220422052400200241786a4280808080d0a0fdf000370300200241706a4200370300200241686a420037030020044200370300200041786a4200370300200041706a4200370300200041686a4200370300200342003703002003200410012005220041606a2202220324002000416c6a41edde013b0100200041686a220441e5dc91aa06360200200242c5a48d928386d5b7eb00370300200041786a220220022903004280808080808080801c84220637030020042903002107200041706a29030021082003220041606a220322022400200241606a220422052400200241786a2006370300200241706a2008370300200241686a2007370300200442c5a48d928386d5b7eb00370300200041786a42808080808080808001370300200041706a4200370300200041686a4200370300200342003703002003200410012005220041606a220222032400200041626a41c4003a0000200241c5a8013b0100200041786a220420042903004280808080808080800684220637030020022903002107200041686a2903002108200041706a29030021092003220041606a220322022400200241606a220422052400200241786a2006370300200241706a2009370300200241686a200837030020042007370300200041786a42808080808080808002370300200041706a4200370300200041686a4200370300200342003703002003200410012005220041606a22022203240020021002200141a8016a4200200229030022064220862006422088200041686a290300220642208684200041706a350200422086200642208884100920014188016a20012903a801200141b0016a290300200141b8016a3502004200100920014190016a290300210620014198016a2903002107200141a0016a290300210820012903880121092003220041406a220222032400200041786a42808080808080808003370300200041706a4200370300200041686a4200370300200041606a4200370300200041586a2008370300200041506a2007370300200041486a2006370300200220093703002001420037038002200142003703f801200142808080103e0288021003200141f8016a200241c00010041a200141d8016a410041201005200141e8006a20012903d801200141e0016a290300200141e8016a290300200141f0016a290300100920014180016a2903002106200141f8006a2903002107200141f0006a2903002108200129036821092003220041606a220322022400200241606a220422052400200041786a4200370300200041706a4200370300200041686a420037030020034200370300200320041006200141c8006a2004290300200241686a290300200241706a290300200241786a2903001009200141286a20092008200720061009200141306a2903002106200141386a2903002107200141406b290300210820012903282109200141086a2001290348200141d0006a290300200141d8006a290300200141e0006a2903001009200141106a290300210a200141186a290300210b200141206a290300210c2001290308210d2005220041606a220322022400200241606a22042400200241786a200c370300200241706a200b370300200241686a200a3703002004200d370300200041786a2008370300200041706a2007370300200041686a20063703002003200937030020032004100120014190026a24000f0b41800841171007000b0c00100a41970841c42510080b0be32501004180080bdb2546756e6374696f6e206973206e6f742070617961626c650061736d0100000001460b60027f7f0060017f0060037f7f7f0060000060077f7f7f7f7f7f7f0060047f7e7e7e0060057f7e7e7e7e0060077e7e7e7e7e7e7e006000017f60047e7f7f7f017f6000017e0289020c08657468657265756d0c67657443616c6c56616c7565000108657468657265756d0a6765744761734c656674000a08657468657265756d0a63616c6c537461746963000908657468657265756d0e72657475726e44617461436f7079000208657468657265756d0b73746f726167654c6f6164000008657468657265756d06726576657274000008657468657265756d0967657443616c6c6572000108657468657265756d0c73746f7261676553746f7265000008657468657265756d036c6f67000408657468657265756d0f67657443616c6c4461746153697a65000808657468657265756d0c63616c6c44617461436f7079000208657468657265756d0666696e69736800000305040605070305030100020608017f0141f088040b071102066d656d6f72790200046d61696e000f0ac321048d0300200020044228884280fe03832004421888428080fc0783200442088842808080f80f8320044208864280808080f01f832004421886428080808080e03f83200442288642808080808080c0ff00832004423886200442388884848484848484370300200041186a20014228884280fe03832001421888428080fc0783200142088842808080f80f8320014208864280808080f01f832001421886428080808080e03f83200142288642808080808080c0ff00832001423886848484848484200142388884370300200020024228884280fe03832002421888428080fc0783200242088842808080f80f8320024208864280808080f01f832002421886428080808080e03f83200242288642808080808080c0ff00832002423886848484848484200242388884370310200020034228884280fe03832003421888428080fc0783200342088842808080f80f8320034208864280808080f01f832003421886428080808080e03f83200342288642808080808080c0ff008320034238868484848484842003423888843703080baf0402057f017e230041d0016b22052400200522044188016a100020042903880120044190016a29030084500440200441e8006a20012002200342ffffffff0f834200100c200441f0006a2903002101200441f8006a290300210220044180016a290300210320042903682109200541406a220622072400200541786a42808080808080808003370300200541706a4200370300200541686a4200370300200541606a4200370300200541586a2003370300200541506a2002370300200541486a200137030020062009370300200442003703c001200442003703b801200442808080103e02c8011001200441b8016a200641c00010021a20044198016a410041201003200441c8006a200429039801200441a0016a290300200441a8016a290300200441b0016a290300100c200441286a2004290348200441d0006a290300200441d8006a290300200441e0006a290300100c200441306a2903002101200441386a2903002102200441406b2903002103200429032821092007220541606a220722062400200641606a22082400200541786a2003370300200541706a2002370300200541686a200137030020072009370300200720081004200441086a2008290300200641686a290300200641706a290300200641786a290300100c200441106a2903002101200441186a290300210220042903082103200041186a200441206a290300370300200020023703102000200137030820002003370300200441d0016a24000f0b41d90841171005000bec1502077f0b7e230041d0056b22082400200822074188056a1000024002400240024020072903880520074190056a29030084500440200841606a2209220b240020091006200741e8046a42002009290300220e422086200e422088200841686a290300220e42208684200841706a350200422086200e42208884100c200741c8046a20072903e804200741f0046a290300200741f8046a3502004200100c200741d0046a290300210e200741d8046a290300210f200741e0046a290300211020072903c8042112200b220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2010370300200841506a200f370300200841486a200e37030020092012370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a410041201003200741a8046a200729039805200741a0056a220b290300200741a8056a220c290300200741b0056a220d290300100c200741c0046a2903002111200741b8046a2903002113200741b0046a290300211420072903a8042115200a220841606a2209220a24002009100620074188046a42002009290300220e422086200e422088200841686a290300220e42208684200841706a350200422086200e42208884100c200741e8036a20072903880420074190046a29030020074198046a3502004200100c200741f0036a290300210e200741f8036a290300210f20074180046a290300211020072903e8032112200a220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2010370300200841506a200f370300200841486a200e37030020092012370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a410041201003200741c8036a200729039805200b290300200c290300200d290300100c200741a8036a20072903c803200741d0036a290300200741d8036a290300200741e0036a290300100c200741b0036a290300210e200741b8036a290300210f200741c0036a290300211020072903a8032112200a220841606a220a22092400200941606a220c220d2400200841786a2010370300200841706a200f370300200841686a200e370300200a2012370300200a200c100420074188036a200c290300200941686a290300200941706a290300200941786a290300100c200741a0036a290300210e20074198036a290300210f20074190036a2903002110200729038803211220074198056a1000200729039805200b290300844200520d0120122003542208201020045420042010511b2209200f200554220b200e2006542006200e511b2005200f852006200e8584501b0d02200741e8026a2015201420132011100c200741f0026a2903002111200741f8026a290300211320074180036a290300211420072903e8022115200741c8026a201220037d201020047d2008ad7d200f20057d220f2009ad22107d200e20067d200bad7d200f201054ad7d100c200741d0026a290300210e200741d8026a290300210f200741e0026a290300211020072903c8022112200d220841606a220b22092400200941606a220a220c2400200941786a2010370300200941706a200f370300200941686a200e370300200a2012370300200841786a2014370300200841706a2013370300200841686a2011370300200b2015370300200b200a1007200741a8026a20002001200242ffffffff0f834200100c200741b0026a2903002110200741b8026a2903002112200741c0026a290300210020072903a8022101200c220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2000370300200841506a2012370300200841486a201037030020092001370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a41004120100320074188026a200729039805200741a0056a220b290300200741a8056a220c290300200741b0056a220d290300100c200741a0026a290300211420074198026a290300211520074190026a29030021162007290388022117200a220841406a2209220a2400200841786a42808080808080808003370300200841706a4200370300200841686a4200370300200841606a4200370300200841586a2000370300200841506a2012370300200841486a201037030020092001370300200742003703c005200742003703b805200742808080103e02c8051001200741b8056a200941c00010021a20074198056a410041201003200741e8016a200729039805200b290300200c290300200d290300100c200741c8016a20072903e801200741f0016a290300200741f8016a29030020074180026a290300100c200741d0016a290300210e200741d8016a290300210f200741e0016a290300210220072903c8012111200a220841606a220a22092400200941606a220c220d2400200841786a2002370300200841706a200f370300200841686a200e370300200a2011370300200a200c1004200741a8016a200c290300200941686a290300200941706a290300200941786a290300100c200741c0016a290300210e200741b8016a290300210f200741b0016a290300210220072903a801211120074198056a1000200729039805200b290300844200520d03200320117c221820115422082008ad200220047c7c221320025420022013511b22082005200f7c22112008ad7c2202200f542002201154ad2011200f54ad2006200e7c7c7c2211200e54200e2011511b2002200f85200e20118584501b0d0420074188016a2017201620152014100c20074190016a290300210e20074198016a290300210f200741a0016a29030021142007290388012115200741e8006a2018201320022011100c200741f0006a2903002102200741f8006a290300211120074180016a290300211320072903682116200d220841606a220b22092400200941606a220a220c2400200941786a2013370300200941706a2011370300200941686a2002370300200a2016370300200841786a2014370300200841706a200f370300200841686a200e370300200b2015370300200b200a1007200c220841606a2209220b240020091006200741c8006a42002009290300220e422086200e422088200841686a290300220e42208684200841706a350200422086200e42208884100c200741d0006a290300210e200741d8006a350200210f20072903482102200b220841606a2209220b2400200741286a2002200e200f4200100c200741306a290300210e200741386a290300210f20072903282102200841786a200741406b290300370300200841706a200f370300200841686a200e37030020092002370300200b220841606a220b220a2400200841786a2000370300200841706a2012370300200841686a2010370300200b2001370300200a220841606a220a2400200741086a2003200420052006100c200741106a2903002106200741186a290300210420072903082105200841786a200741206a290300370300200841706a2004370300200841686a2006370300200a2005370300200a412041034180082009200b41001008200741d0056a24000f0b41d90841171005000b41d90841171005000b41bb08411e1005000b41d90841171005000b41a008411b1005000b920402047f047e230041a0016b2200210120002400024002400240100941034d0d00200041706a220022022400200041004104100a2000280200220041a98bf0dc7b460d01200041f0c08a8c03470d002002220041606a220222032400200241044120100a200141406b2002290300200041686a290300200041706a290300200041786a290300100c200141206a2001290340200141c8006a290300200141d0006a350200100d20012001290320200141286a290300200141306a290300200141386a290300100c200141086a2903002104200141106a2903002105200141186a2903002106200129030021072003220041606a22022400200041786a2006370300200041706a2005370300200041686a20043703002002200737030020024120100b0c020b41004100100b0c010b2002220041406a2202220324002002410441c000100a20014180016a2002290300200041486a290300200041506a290300200041586a290300100c200141e0006a200041606a290300200041686a290300200041706a290300200041786a290300100c20012903800120014188016a29030020014190016a3502002001290360200141e8006a290300200141f0006a290300200141f8006a290300100e2003220041606a22022400200041786a42808080808080808001370300200041706a4200370300200041686a42003703002002420037030020024120100b0b200141a0016a24000b0b7701004180080b70ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef536166654d6174683a206164646974696f6e206f766572666c6f77536166654d6174683a207375627472616374696f6e206f766572666c6f7746756e6374696f6e206973206e6f742070617961626c65"
})
"0x008d8ab4d995cdef95fcc61b0d009470b7a69f6b3d37e0d76734bcacf62319fe"
```

Note the return value from the `sendTransaction` is the transaction hash. We can use it to find the address of the newly created contract.

```text
// USE THE TX HASH FROM ABOVE
> cmt.getTransactionReceipt("0x008d8ab4d995cdef95fcc61b0d009470b7a69f6b3d37e0d76734bcacf62319fe")
{
  blockHash: "0xff8aabe3b70ea6796cc1b945b10d717d462ae24c49f8e05dbec2dc2f4511790f",
  blockNumber: 27978,
  contractAddress: "0x70f94d58cc3fdcbeac7140f35a087da9fcd09b94",
  cumulativeGasUsed: 1519352,
  from: "0x7eff122b94897ea5b0e2a9abf47b86337fafebdc",
  gasUsed: 1519352,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: null,
  transactionHash: "0x8b102fb7744f3472328131f921efdbad3a9ef3c6225c682196ad2a3a8bb34a6e",
  transactionIndex: 0
}
// SET THE CONTRACT ADDRESS
> contractAddress = "0x70f94d58cc3fdcbeac7140f35a087da9fcd09b94"
```

Next, create an instance of this contract at the above address. You will need the ABI for this contract \(based on the Solidity source code\) for this to work. Then, we can query the account balances in this ERC20 token contract. The creator address of this contact has all the initial supply of the tokens, and everyone else is zero.

```text
> abi = [{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"constant":true,"inputs":[{"name":"account","type":"address"}],"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"to","type":"address"},{"name":"amount","type":"uint256"}],"name":"transfer","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"anonymous":false,"inputs":[{"indexed":true,"name":"_from","type":"address"},{"indexed":true,"name":"_to","type":"address"},{"indexed":false,"name":"_value","type":"uint256"}],"name":"Transfer","type":"event"}]
> contract = cmt.contract(abi);
> contractInstance = contract.at(contractAddress);
> contractInstance.balanceOf.call("0x7eff122b94897ea5b0e2a9abf47b86337fafebdc");
100000000
> contractInstance.balanceOf.call("0x77beb894fc9b0ed41231e51f128a347043960a9d");
0
```

Finally, we can make `transfer` calls on the contract to transfer ERC20 tokens between accounts.

```text
> contractInstance.transfer.sendTransaction(
    "0x77beb894fc9b0ed41231e51f128a347043960a9d",
    123, 
    {
        from: "0x7eff122b94897ea5b0e2a9abf47b86337fafebdc",
        gas: 5000000
    }
);
> contractInstance.balanceOf.call("0x7eff122b94897ea5b0e2a9abf47b86337fafebdc");
99999877
> contractInstance.balanceOf.call("0x77beb894fc9b0ed41231e51f128a347043960a9d");
123
```

That's it. You have now deployed and executed an ewasm smart contract on a real blockchain. Welcome to the future of Ethereum!
