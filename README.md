![](lucky.gif)

# Golf on Mt. Fuji&nbsp;&nbsp;⛳&nbsp;🗻&nbsp;🏌️

By Kim Slawson, February – March 2021

*NOMAM* 2021 10-Line BASIC competition entry (EXTREME-256 category)

Requires an Atari 8-bit machine or emulator running TurboBASIC XL (800XL minimum)

The provided disk image contains TurboBASIC XL and automatically runs the game.

*Update (2022-05-12):* [Now you can play the game right in your browser](https://mrk.sed.pl/bevy-atari/#disk_1=https://slawson.org/atari/fujigolf/fujigolf.atr) in the excellent [bevy-atari](https://github.com/mrk-its/bevy-atari) emulator! (Use the arrow keys and the shift key)

## The pitch&nbsp;&nbsp;📈
18 holes of golf, but on Japan’s famous Mount Fuji. 

## To play&nbsp;&nbsp;🕹️➡️🕳️
Use the joystick to aim the cursor in the direction you want to hit the ball. The farther you aim, the harder you hit the ball. Hit the ball into the hole in as few strokes as possible. Don’t forget to turn up your sound!

## Please note&nbsp;❗
“Mount Fuji” is procedurally generated. Any similarity to actual mountains, living or dead, is purely coincidental.

## Colophon&nbsp;&nbsp;🧰
Golf on Mount Fuji would not have been possible without standing on the shoulders of giants. I owe a large debt of gratitude to the following AtariAge users: @dmsc for basicParser, @skr for MacTurban, @luckybuck for Sublime Text integration and @mozzwald and @tschak909 for the wonderful FujiNet adapter (ALL HAIL THE FUJINET!) 

## Source code&nbsp;&nbsp;💻
Please find the commented source code below (if you are curious). You may also wish to look at the obfuscated source code (if you are masochistic).
<details>
 <summary><strong>Commented source</strong></summary>
 <p>

```basic
REM FUJIGOLF - Golf on Mount Fuji
REM built with Turban (TURboBAsic Nifty)
REM obsfuscated using tbxlparser
REM requires Turbo-BASIC XL
REM by Kim Slawson
REM 2021-02-23

REM set up graphics mode, draw title, turn off cursor
GR.0:COL.20:TE.4,0,"GOLF":POS.14,8:POKE 752,1:?"ON MT. FUJI"

REM get screen location -- like scrlo=PEEK(88):scrhi=PEEK(89) but better
scrmem=DPEEK(88)

REM set up display list
DL=PEEK(560)+PEEK(561)*256
REM read in assembly code to modify graphics
FOR I=0 TO 42:READ B:POKE 1536+I,B:NEXT I
DATA 72,152,72,172,0,4,185,2,4,141,10,212,141,24,208,238,0,4,173,0,4,205,1,4,144,5,169,0,141,0,4,104,168,104,64,169,192,141,14,212,76,98,228
REM modify display list for interrupt
FOR I=6 TO 27:POKE DL+I,PEEK(DL+I)+128:NEXT I
FOR I=2 TO 3:POKE DL+I,PEEK(DL+I)+128:NEXT I
REM read in background color data starting at mem loc 1024
POKE 1024,0:POKE 1025,24
FOR I=0 TO 23:READ B:POKE 1026+I,B:NEXT I
DATA 128,96,82,68,54,40,26,236,192,194,194,196,196,196,196,198,198,198,198,198,198,198,198,198
REM point to DLI on page 6 and enable interrupt
POKE 512,0:POKE 513,6
REM disable/re-enable deferred VBLANK
POKE 66,1:POKE 548,35:POKE 549,6:POKE 66,0

REM Don't forget to set luminance!
POKE 709,15

REM need an array for heights of each bit of ground
DIM g(40)

REM main game loop for all 18 holes
FOR hole=1 TO 18
  REM turn off the cursor
  POKE 752,1

  REM Q: how long are straight runs? 
  REM A: shorter in the middle of the mountain, longer at the ends
  long=ABS(9-hole)
  REM starting height of ground for this hole (relative to which hole we're on). 
  REM It's a mountain so it's higher in the middle
  starting=9-long 
  REM y location of top of the ground - can't go above this
  REM (don't want an out-of-bounds error or an impossible level) 
  top=12-starting
  REM set some initial conditions
  x=0:ball=0:y=22-starting 

  REM draw the ground
  WHILE x<40
    REM the slope is procedurally determined by the hole number and some tricky math
    s=INT((RND*3-1)+0.1*(hole-9.5))
    REM because ATASCII is simple and we're just using {udiag} and {ddiag} we need to limit the slope
    IF s<-1:S=-1:ENDIF:IF S>1:S=1:ENDIF
    REM let's figure out how high the ground is based on the slope, within limits
    y=y+s:IF Y<top:Y=top:s=0:ENDIF:IF Y>22:Y=22:s=0:ENDIF
    REM set the ground array to the correct height
    REM we have to keep track so we know when the ball lands
    g(x)=y
    REM use either {udiag} (atascii 8) or {ddiag} (atascii 10) 
    REM but they're separated by a character in the middle so we need the correct offset
    REM (note this is using the internal representation, where the control characters 72 and 74 are between
    REM  the uppercase and lowercase letters. Equivalents would be CHR$(8) and CHR$(10) )
    c=73+s

    REM the Earth isn't flat, but sometimes golf courses are.
    REM this plots a straight run for the amount appropriate to the given hole
    REM and sets the ground array to the correct height for each spot
    IF s=0
      REM put the ball on the first flat spot in the level
      IF ball=0:COL.20:ball=1:bx=x:by=y-1:PLOT bx,by:ENDIF
      FOR i=0 to long
        IF x+i=40:EXIT:ENDIF
        g(x+i)=y
        POKE scrmem+(x+i)+y*40,128
      NEXT i
      x=x+long
    ENDIF
    REM if we're going up or down slope, pick the correct character, 
    REM poke it into screen mem, and set the height of the ground array.
    IF s=-1:POKE scrmem+x+y*40,c:ENDIF
    IF s=1:POKE scrmem+x+(y-1)*40,c:g(x)=y-1:ENDIF
    REM The y-1 corrects a bug where the ball lands one below the ground on a downslope

    x=x+1
  WEND

  REM fill the ground
  COLOR 160:PAINT 1,23

  REM set the color left behind by the ball, so as to be able to redraw it later
  oc=32

  rem draw the hole. Set the hole depth 1 lower than surroundings so do -1
  COLOR 32:PLOT 33,g(33)-1:COLOR 95:PLOT 33,g(33): REM plot _
  g(33)=g(33)+1

  REM now is the time for all good players to make a hole in one 
  REM turn on cursor for swing setup. Aim using cursor.
  poke 752,0
  REPEAT

    REM get joystick position and remember the current aiming vectors
    S=STICK(0):oh=horiz:ov=vert
    REM aiming vector is relative to ball position, 
    REM magnitude determined by the position of the cursor
    vert=vert+((S=13)-(S=14)):horiz=horiz+((S=7)-(S=11))

    #p
    REM cursor goes one below position when POSitioning and PRINTing so do -1 
    REM (else it's an off-by-one and a possible out-of-bounds off the y axis)
    posx=bx+horiz:posy=by+vert-1
    REM check if relative position is out of bounds
    REM if yes, restore velocity to previous value and jump back 
    IF posx<0 OR posx>39:horiz=oh:GO# p:ENDIF
    REM we don't want the ball being hit downward because that doesn't make sense
    IF posy<0 OR posy>by-1:vert=ov:GO# p:ENDIF
    REM put the cursor where the player aims
    POS.posx,posy:?CHR$(29);
    pause 5:REM Goldilocks timing for aiming. Not too slow, but not too fast.
    POKE 77,0: REM disable attract mode periodically ... 
    REM ... cuz it's ugly and also doesn't play well with the DLI

    REM use the built-in keyboard handler to make the stroke noise
    IF(STRIG(0)=0)
      OPEN #1,4,0,"K:":IRQEN=PEEK(16)
      POKE 16,0:POKE 53774,0:POKE 764,12
      GET #1,C:POKE 16,IRQEN:POKE 53774,IRQEN
      CLOSE #1

      REM keep track of strokes and score
      stroke=stroke+1
      POKE 752,1:POS.0,23:?"Hole ";hole;", stroke ";stroke;", score ";score+stroke;

      REM if the player fumbles their putt by wasting their shot, 
      REM then they get a stroke penalty. Too bad, so sad.
      IF vert>0:GO# l:ENDIF

      REM set up some physics for parabolic motion of the ball
      t=0:xi=bx:yi=by

      REPEAT:REM ball is in flight
        pause 10:REM slow it down so the flight seems natural

        REM time passes. Remember the ball's position. Calculate the next position.
        t=t+1:ox=bx:oy=by:bx=bx+horiz/2:by=by+vert:vert=vert+3
        
        REM prevent out-of-bounds errors to the left and the right
        IF bx<0:bx=0:ENDIF
        IF bx>38:bx=38:ENDIF
        
        REM “Houston, Tranquility Base Here. The Eagle has Landed." (the ball hit the ground)
        IF by>=g(bx):by=g(bx)-1:landed=1:ENDIF

        REM prevent out-of-bounds errors to the top
        IF by<0:by=0:ENDIF

        REM redraw the old scenery behind the ball and draw the ball at its new position
        COLOR oc:PLOT ox,oy:LOCATE bx,by,oc:COLOR 20:PLOT bx,by
      UNTIL landed

      REM should the ball roll? nah. we don't have enough lines for REAL physics ¯\_(ツ)_/¯
      #l:landed=0:horiz=0:vert=0:REM bx=int(bx)
      POKE 752,0:REM toggle the cursor
    ENDIF

  REM keep letting the player take shots until the ball lands in the hole
  UNTIL (32<bx) AND (bx<34)  

  REM make a "ball falling in the hole and bouncing noise" by clicking the speaker
  REM the period of the noise is based on how far the ball flew
  i=ABS(oh):REPEAT:PAUSE I:Poke 53279,1:I=I/1.25:UNTIL I<1

  REM OMG the player just got a hole in one 111111! !!!! eleventy-one!!1111
  REM Reward them for their good work and play a nice pleasant chord building over time
  IF stroke=1:TEXT 4,0,"HOLE":TEXT 4,8,"IN 1":SOUND 0,81,10,3:PAUSE 25:SOUND 1,64,10,7:PAUSE 25:sound 2,53,10,11:pause 25:sound 3,45,10,15:pause 200:SOUND:pause 100:ENDIF : rem HOLE IN ONE!

  REM Add the strokes to the score. Reset the stroke counter for the next hole.
  score=score+stroke:stroke=0

  REM turn off the cursor and clear the screen
  POKE 752,1:CLS
NEXT hole

REM display final score, encourage replay
POS.2,8:?"YOUR FINAL GOLF ON MT. FUJI SCORE IS"
COLOR 20:TEXT 12,9,score
PAUSE 200:POS.13,19:?"FIRE TO REPLAY"
REM wait for user and disable attract mode
WHILE STRIG(0):POKE 77,0:WEND:RUN
```
</details>

<details>
 <summary><strong>Obfuscated BASIC tenliner source</strong></summary>
 <p>

```basic
0AA=-%1:A8=%1+%3:AB=%2*%3:AD=%2*A8:AC=%1+AD:A7=%1+AC:AE=%2*AB:A9=%2*A7:A4=%1+AE+A9:A6=%2*A9:A5=752:GR.%0:C.A9:TE.A8,%0,"GOLF":POS.14,AD:POKEA5,%1:?"ON MT. FUJI":A=DPEEK(88):B=PEEK(561)*256+PEEK(560):F.C=%0TO42:READD:POKE1536+C,D:N.C
1D.72,152,72,172,0,4,185,2,4,141,10,212,141,24,208,238,0,4,173,0,4,205,1,4,144,5,169,0,141,0,4,104,168,104,64,169,192,141,14,212,76,98,228
2F.C=AB TO27:POKEB+C,PEEK(B+C)+128:N.C:F.C=%2TO%3:POKEB+C,PEEK(B+C)+128:N.C:POKE1024,%0:POKE1025,24:F.C=%0TO23:READD:POKE1026+C,D:N.C:D.128,96,82,68,54,40,26,236,192,194,194,196,196,196,196,198,198,198,198,198,198,198,198,198
3POKE512,%0:POKE513,AB:POKE66,%1:POKE548,35:POKE549,AB:POKE66,%0:POKE709,15:DIMA(A6):F.E=%1TO18:POKEA5,%1:F=ABS(AC-E):G=AC-F:H=AE-G:I=%0:J=%0:K=22-G:W.I<A6:L=INT(RND*%3-%1+(E-9.5)*.1):IFL<AA:L=AA:END.:IFL>%1:L=%1:END.
4K=K+L:IFK<H:K=H:L=%0:END.:IFK>22:K=22:L=%0:END.:A(I)=K:M=73+L:IFL=%0:IFJ=%0:C.A9:J=%1:N=I:O=K-%1:PL.N,O:END.:F.C=%0TOF:IFI+C=A6:EX.:END.:A(I+C)=K:POKEI+C+A+K*A6,128:N.C:I=I+F:END.:IFL=AA:POKEA+I+K*A6,M:END.:IFL=%1:POKEA+I+(K-%1)*A6,M:A(I)=K-%1
5END.:I=I+%1:WE.:C.160:PAI.%1,23:P=32:C.32:PL.A4,A(A4)-%1:C.95:PL.A4,A(A4):A(A4)=A(A4)+%1:POKEA5,%0:REP.:L=STICK(%0):Q=R:S=T:T=(L=13)-(L=14)+T:R=(L=7)-(L=11)+R
6#A:U=N+R:V=O+T-%1:IFU<%0ORU>39:R=Q:GO#A:END.:IFV>O-%1ORV<%0:T=S:GO#A:END.:POS.U,V:?"^]";:PA.5:POKE77,%0:IFSTRIG(%0)=%0:O.#%1,A8,%0,"K:":W=PEEK(16):POKE16,%0:POKE53774,%0:POKE764,AE:GET#%1,M:POKE16,W:POKE53774,W:CL.#%1:X=X+%1:POKEA5,%1:POS.%0,23
7?"Hole ";E;", stroke ";X;", score ";Y+X;:IFT>%0:GO#B:END.:Z=%0:_=N:A0=O:REP.:PA.A7:Z=Z+%1:A1=N:A2=O:N=R/%2+N:O=O+T:T=T+%3:IFN<%0:N=%0:END.:IFN>38:N=38:END.:IFO>=A(N):O=A(N)-%1:A3=%1:END.:IFO<%0:O=%0:END.:C.P:PL.A1,A2:LOC.N,O,P:C.A9:PL.N,O:U.A3
8#B:A3=%0:R=%0:T=%0:POKEA5,%0:END.:U.32<N ANDN<34:C=ABS(Q):REP.:PA.C:POKE53279,%1:C=C/1.25:U.C<%1:IFX=%1:TE.A8,%0,"HOLE":TE.A8,AD,"IN 1":SO.%0,81,A7,%3:PA.25:SO.%1,64,A7,7:PA.25:SO.%2,53,A7,11:PA.25:SO.%3,45,A7,15
9PA.200:SO.:PA.100:END.:Y=Y+X:X=%0:POKEA5,%1:CLS:N.E:POS.%2,AD:?"YOUR FINAL GOLF ON MT. FUJI SCORE IS":C.A9:TE.AE,AC,Y:PA.200:POS.13,19:?"FIRE TO REPLAY":W.STRIG(%0):POKE77,%0:WE.:RUN
```
</details>

## Get in touch&nbsp;&nbsp;📩

  * [OxC0FFEE on AtariAge](https://atariage.com/forums/profile/50996-oxc0ffee/)
  * [OxC0FFEE on Twitter](https://twitter.com/OxC0FFEE)
  * [kimslawson@gmail.com](mailto:kimslawson@gmail.com)
