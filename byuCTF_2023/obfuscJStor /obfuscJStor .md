# byuCTF 2023 - obfuscJStor (Rev)

__Description:__

> Obfuscated JavaScript?? Really??

There was the following obfuscated JS code:

```js
function _0x12de(){var _0x6ab222=['\x2e\x69\x6f','\x75\x73\x63\x61\x74','\x37\x32\x4f\x4f\x6e\x7a\x73\x4d','\x61\x5f\x74\x6f\x6f','\x6c\x6f\x67','\x62\x79\x75\x63\x74','\x32\x30\x35\x38\x31\x31\x31\x56\x73\x4a\x6d\x4e\x74','\x64\x61\x79\x73\x5f','\x35\x62\x6b\x68\x53\x6b\x77','\x36\x32\x37\x38\x77\x53\x77\x45\x56\x49','\x31\x32\x35\x33\x31\x33\x30\x78\x4e\x74\x74\x57\x77','\x48\x6d\x6d\x6d\x6d','\x6c\x5f\x74\x6f\x5f','\x77\x68\x65\x72\x65','\x66\x6c\x61\x67\x20','\x34\x31\x30\x35\x39\x34\x34\x58\x71\x67\x53\x54\x64','\x31\x30\x69\x47\x78\x53\x78\x74','\x35\x33\x4d\x50\x56\x43\x43\x73','\x63\x61\x74\x6f\x72','\x6d\x61\x6b\x65\x5f','\x6f\x62\x66\x75\x73','\x32\x35\x34\x30\x30\x39\x74\x71\x59\x51\x79\x6b','\x35\x31\x30\x35\x30\x31\x46\x57\x64\x52\x56\x71','\x66\x7b\x6f\x6e\x65','\x64\x65\x6f\x62\x66','\x68\x65\x73\x65\x5f','\x5f\x6f\x66\x5f\x74','\x31\x37\x32\x36\x34\x35\x6f\x6b\x76\x58\x66\x70','\x69\x73\x3f','\x34\x6d\x6f\x71\x49\x6c\x56'];_0x12de=function(){return _0x6ab222;};return _0x12de();}(function(_0x2a4cef,_0x9e205){var _0x539a11=_0x2a7d,_0x40cc8a=_0x2a4cef();while(!![]){try{var _0x2d47a2=-parseInt(_0x539a11(0x1f1))/0x1*(-parseInt(_0x539a11(0x207))/0x2)+parseInt(_0x539a11(0x1f6))/0x3*(parseInt(_0x539a11(0x1fd))/0x4)+-parseInt(_0x539a11(0x206))/0x5*(-parseInt(_0x539a11(0x208))/0x6)+-parseInt(_0x539a11(0x1f5))/0x7*(parseInt(_0x539a11(0x200))/0x8)+parseInt(_0x539a11(0x204))/0x9*(-parseInt(_0x539a11(0x1f0))/0xa)+parseInt(_0x539a11(0x1fb))/0xb+parseInt(_0x539a11(0x1ef))/0xc;if(_0x2d47a2===_0x9e205)break;else _0x40cc8a['push'](_0x40cc8a['shift']());}catch(_0x4063a2){_0x40cc8a['push'](_0x40cc8a['shift']());}}}(_0x12de,0x54f50));function _0x2a7d(_0x339bb1,_0x1a0657){var _0x12def3=_0x12de();return _0x2a7d=function(_0x2a7d9a,_0x2b9202){_0x2a7d9a=_0x2a7d9a-0x1ee;var _0x34fb38=_0x12def3[_0x2a7d9a];return _0x34fb38;},_0x2a7d(_0x339bb1,_0x1a0657);}function hi(){var _0x398601=_0x2a7d;document['\x64\x6f\x6d\x61\x69'+'\x6e']==_0x398601(0x1f4)+_0x398601(0x1f2)+_0x398601(0x1fe)&&console[_0x398601(0x202)](_0x398601(0x203)+_0x398601(0x1f7)+_0x398601(0x1fa)+_0x398601(0x1f9)+_0x398601(0x205)+'\x69\x6d\x6d\x61\x5f'+_0x398601(0x1f3)+_0x398601(0x201)+_0x398601(0x20a)+_0x398601(0x1f8)+_0x398601(0x1ff)+'\x65\x5f\x74\x68\x69'+'\x73\x7d'),console['\x6c\x6f\x67'](_0x398601(0x209)+'\x20\x49\x20\x77\x6f'+'\x6e\x64\x65\x72\x20'+_0x398601(0x20b)+'\x20\x74\x68\x65\x20'+_0x398601(0x1ee)+_0x398601(0x1fc));}hi();
```

Via an online deobfuscation [tool](https://deobfuscate.relative.im/) I got the deobfuscated source code and the flag.

__Deobfuscated Code:__

```js
function hi() {
  document.domain == 'obfuscator.io' &&
    console.log(
      'byuctf{one_of_these_days_imma_make_a_tool_to_deobfuscate_this}'
    )
  console.log('Hmmmm I wonder where the flag is?')
}
hi()
```

Flag: `byuctf{one_of_these_days_imma_make_a_tool_to_deobfuscate_this}`
