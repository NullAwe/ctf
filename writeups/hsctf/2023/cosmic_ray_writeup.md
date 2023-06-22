{\rtf1\ansi\ansicpg1252\cocoartf2580
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 Looking at `data.txt` and the program, realize that two cache numbers being very close together is very helpful.\
\
We also realize that the last number (which has since been turned into a `0`) in the cache must be `c` (based off the program).\
\
Let the last nonzero element in the cache be `a` and the last element be `b` (`b` = `c`). Our cache consists of:\
```\
[\'85, a, 0, 0, 0, 0, 0, 0, b]\
```\
\
We now examine the minimum and maximum possible value of the numbers following `a`. Follwing the program logic, these values are:\
```\
minimum: [a, a^2,   a^4,     a^8,     a^16,      a^32,      a^64,      a^128]\
maximum: [a, a^2*m, a^4*m^3, a^8*m^7, a^16*m^15, a^32*m^31, a^64*m^63, a^128*m^127]\
```\
Therefore, `b = a^128 * m^x (mod n)`, where `x` is in the range `[0, 128)`.\
Realize that this information isn\'92t enough, because even if we brute forced the exponent on `m`, we can\'92t reverse it to get anything useful.\
\
We have only used one number from `cache`. If we look at the second-smallest gap of `0`s, we have a similar situation (a gap of length `13`), so something like:\
```\
[\'85, a2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, b2, \'85]\
```\
Similar to earlier, we have `b2 = a2^(2^13) * m^x2 (mod n)`, where `x2` is in the range `[0, 2^13)`.\
\
Now we can brute force over both these ranges. We can iterate `i` over range `[0, 128 = 2^7)` and `j` over range `[0, 2^13)`, and we want to test if these exponents will work as `x` and `x2`.\
\
To do this, we take our two equations:\
1. `b = a^128 * m^x (mod n)`\
2. `b2 = a2^(2^13) * m^x2 (mod n)`\
which translates to:\
1. `m^x = b * (a^128)^(-1) (mod n)`\
2. `m^x2 = b2 * (a2^(2^13))^(-1) (mod n)`\
\
We can check if these exponents work by raising equation (1) to the `x2` power and equation (2) to the `x` power. Since the right-hand sides of both equations are known, we can simply check if the right-hand sides of the equations are equal after this operation.\
\
We can run this brute-force script to find `x` and `x2`:\
```py\
import Crypto.Util.number\
\
def toString(num):\
    return Crypto.Util.number.long_to_bytes(num)\
\
n = 471794602312181068011601975537791353222470509157304225172118054851894222850510156945514492360273864411886365421734158872203071445682080353947715317468911391989193241590893667789460995360376512456430107837622910114992871899389148304762185977527005056570902340546791024234066680504170924452971514521601\
\
a = 343190004788499437607845898284239695113462670688664914367031457496526965266002656031194988783696950107365971573480119334073126431699832428382021510495753660712879273504276822902473320337174071411539017073199488942856737279581115287981594879580122312473677881860716877392288947947120580533022832366626\
b = 297444805157923579595492659756994763982280962834662933228783834277026667796039103340058990582217798282434798699551705567584809248118471599628162646936955352212054484238384955355256944438218079458834836612366173168573544605559990276904501543490924665506484704110848942208163629878854963232105440104720\
bits = 7\
rhs = b * pow(pow(a, -1, n), 2 ** bits, n) % n\
\
a2 = 334823363824713665077070202012026189923379293024123400788079277410774197047658071653971004436304522266736098331382932182667556494290213766301841109369135445876975343696462435549896884832268918971958940247856038703516590983905465795054223071712800975024487595858748717534729668238007443295659690048965\
b2 = 161871115501970062534351795345828929371256682904562729930683298498471934529621287918421582081293144569781197640007409867100986869101025553277338171413324126180647815592159530685007983731537469593377648707222034769451336325256873454740359493423577485057616201643076623733112144282219709887572771424512\
bits2 = 13\
rhs2 = b2 * pow(pow(a2, -1, n), 2 ** bits2, n) % n\
\
x = -1\
x2 = -1\
\
for i in range(2 ** bits):\
  for j in range(2 ** bits2):\
    if i == 0 and j == 0:\
      continue\
    if pow(rhs, j, n) == pow(rhs2, i, n):\
      print(i, j)\
      x = i\
      x2 = j\
      break\
  if x >= 0 and x2 >= 0:\
    break\
```\
\
This script yields `x = 23` and `x2 = 6481`. These numbers are relatively prime, so we can find `m^1` using `m^x` and `m^x2`, as follows:\
```py\
p = pow(rhs, 3945, n) # m^90735\
q = pow(rhs2, 14, n)  # m^90734\
\
flag = (p * pow(q, -1, n)) % n\
\
print(toString(flag)) # function defined in the previous code snippet\
```\
which yields the correct flag.}