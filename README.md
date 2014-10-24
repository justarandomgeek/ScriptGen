ScriptGen
=========
Bitcoin Script generator for P2SH transactions. Primarily for weighted multisig.

###SCORE START
Place a starting score tally on altstack. 
```
0 OP_TOALTSTACK
```

###PUBKEY WEIGHTED SIG
Standard Pay-to-Pubkey, but instead of fully verifying the tx, simply add the weight to the tally on altstack.
```
// <sig> or 0 on stack
OP_IFDUP OP_IF 
<pubkey> OP_CHECKSIGVERIFY 
OP_FROMALTSTACK <weight> OP_ADD OP_TOALTSTACK 
OP_ENDIF
```

###PUBKEY HASH WEIGHTED SIG
Standard Pay-to-Pubkey-Hash, but instead of fully verifying the tx, simply add the weight to the tally on altstack.
```
// <sig> <pubkey> or 0 on stack
OP_IFDUP OP_IF 
OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIGVERIFY 
OP_FROMALTSTACK <weight> OP_ADD OP_TOALTSTACK 
OP_ENDIF
```

###MULTISIG WEIGHTED SIG GROUP
Multisig with number of sigs from stack, pubkeys and num pubkeys embedded. Multiply number of sigs by weight, and add to tally on altstack.
```
//  X <sig> ... <sig> <nSigs> or 0 on stack
OP_IFDUP OP_IF 
OP_DUP OP_TOALTSTACK
<pubkey> ... <pubkey> <nKeys>
OP_CHECKMULTISIGVERIFY
OP_FROMALTSTACK 
// Optionally, insert a multiplier here
OP_FROMALTSTACK OP_ADD OP_TOALTSTACK
OP_ENDIF
```

###MULTISIG PUBKEY HASH WEIGHTED SIG GROUP
Multisig with number of sigs from stack, pubkeys and num pubkeys embedded. Multiply number of sigs by weight, and add to tally on altstack.
```
//  X <sig> ... <sig> <nSigs> <pubkey> ... <pubkey> <nKeys> or 0 on stack
OP_IFDUP OP_IF 
<pubkeyhash> ... <pubkeyhash> <nKeyHashes>

//TODO: verify pubkeys, copy nSigs to altstack, and leave stack ready for OP_CMSV
// perhaps build <nKeys> <pubkey> ... <pubkey> on altstack then brign it all back?
// for fixed <nKeys> it's fairly striaght forward, but doesn't allow keeping keys hidden. any dynamic solution?


OP_CHECKMULTISIGVERIFY
OP_FROMALTSTACK // bring nSigs back
// Optionally, insert a multiplier here
OP_FROMALTSTACK OP_ADD OP_TOALTSTACK
OP_ENDIF
```

###MIN SCORE END
Pass the transaction if the score tally on altstack is high enough.
```
OP_FROMALTSTACK <minweight> OP_GREATERTHANOREQUAL OP_VERIFY
```


###Multipliers
Since `OP_MUL` is disabled, multiplying scores for `OP_CMSV` groups requires some more complicated scripts.
```
//<weight> OP_MUL // DISABLED OPCODE :(
OP_DUP OP_ADD //x2
OP_DUP OP_DUP OP_ADD OP_ADD //x3
OP_DUP OP_ADD OP_DUP OP_ADD //x4
OP_DUP OP_2DUP OP_ADD OP_ADD OP_SWAP OP_DUP OP_ADD OP_ADD//x5
OP_DUP OP_DUP OP_ADD OP_ADD OP_DUP OP_ADD //x3x2=x6
OP_DUP OP_DUP OP_DUP OP_ADD OP_ADD OP_DUP OP_ADD OP_ADD //x7

// Numbers much higher than this are impractical without OP_MUL

```
