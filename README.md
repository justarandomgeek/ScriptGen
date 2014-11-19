ScriptGen
=========
Bitcoin Script generator for P2SH transactions. Primarily for weighted multisig.

###SCORE START
Place a starting score tally on altstack. 
```
0 OP_TOALTSTACK
```

```
006B
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

```
7363<pubkey>ad6c<weight>936b68
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

```
736376a9<pubkeyhash>88ad6c<weight>936b68
```

###MULTISIG WEIGHTED SIG GROUP (points-per-sig)
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

```
7363766b <pubkey> ... <pubkey> <nKeys> af6c <multiplier> 6c936b68
```

###MULTISIG WEIGHTED SIG GROUP (points-per-group)
Multisig with number of sigs from stack, pubkeys and num pubkeys embedded. Pass/fail, fixed number of points for the group.
```
//  X <sig> ... <sig> or 0 on stack
OP_IFDUP OP_IF 
<nSigs> <pubkey> ... <pubkey> <nKeys>
OP_CHECKMULTISIGVERIFY
OP_FROMALTSTACK <weight> OP_ADD OP_TOALTSTACK
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

```
TODO
```

###CONTENT WEIGHTED HASH
Award points for providing content that hashes to a given hash.
```
// <pass> or 0 on stack
OP_IFDUP OP_IF 
OP_HASH160 <hash> OP_EQUALVERIFY 
OP_FROMALTSTACK <weight> OP_ADD OP_TOALTSTACK 
OP_ENDIF
```

```
7363a9<hash>886c<weight>936b68
```

###MIN SCORE END
Use this after all verification modules are done. Pull the score from altstack, compare it to the threshold, and pass/fail the transaction depending on that result.
```
OP_FROMALTSTACK <minweight> OP_GREATERTHANOREQUAL OP_VERIFY
```

```
6c <minweight> a269
```



###Multipliers
Since `OP_MUL` is disabled, multiplying scores for `OP_CMSV` groups requires some more complicated scripts.
```
//<weight> OP_MUL // DISABLED OPCODE :(
//<weight> 95

OP_DUP OP_ADD //x2
7693

OP_DUP OP_DUP OP_ADD OP_ADD //x3
76769393

OP_DUP OP_ADD OP_DUP OP_ADD //x4
76937693

OP_DUP OP_2DUP OP_ADD OP_ADD OP_SWAP OP_DUP OP_ADD OP_ADD//x5
766e93937c769393

OP_DUP OP_DUP OP_ADD OP_ADD OP_DUP OP_ADD //x3x2=x6
767693937693

OP_DUP OP_DUP OP_DUP OP_ADD OP_ADD OP_DUP OP_ADD OP_ADD //x7
7676769393769393

// Numbers much higher than this are impractical without OP_MUL

```
