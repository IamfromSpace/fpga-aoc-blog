Trying to figure out open source tools, wow toolchains suck!
Struggle for a long time with simulators
Clash-wavedrom is _amazing_, eliminates the need for visualization from simulators, they can just run the tests
I can still run tests first in ghci by just sampling the test bench
If simulation was hard, synthesis is even harder!
Give into APIO, which is actually pretty great
Originally go for 64 bit, it totally pops the logic limit, ouch!
Go for 32 bit, but it's super super slow.
Originally I think this is because my pins are too spread out (definitely not it).
Looking into how to make FPGAs go faster, learn about pipelining!
This completely shakes up my notion of what makes FPGA fast, I thought that big blocks of custom logic were the speed advantage, it's all instantaneous, right? lol
Start pipelining, especially around those big adds!
Finally learn how to read the critical path out of nextpnr
Realize that _actually_ it's the damn division that's killing me.
Originally considered not outputting ascii and just going for decimal, that would have been dramatically faster
Read up about how to make decimal division fast, find the double dabble alg
Realize I don't have to implement the double dabble alg, someone has done this for me!
We're fast!  Ish.  At least over the 12MHz hurdle that APIO has by default.  Hardware specific division halves the critical path.
Realize that double dabble could be done step by step go even faster.
This is (of course) is already possible by the lib too.
We're FAST!  Finally hitting speeds over 60Mhz, routing is the slow part and seems to be mostly due to fan out?  Good enough result I'm not too worried.
Finally figure how to read utilization
With BCD as a state machine, we now need dramatically fewer logic units
Based on this, take a stab at 64 bit logic, we got it!  And at 50Mhz! and only 50% utilization of the chip.
Ideally wanted a parallel interface for speed, but the most practical way seems to just be to leverage the UART from my dev board.
How the heck do I do UART?  Specifically there's different clocks involved.  In theory, because I can only read in 1.2M characters/s, my logic doesn't have to be as fast if it runs at 1/10th the clock speed--however, that means that when it's time to go parallel, I won't be fast enough!
I don't know how to avoid leaking a "No Data" on input and output into all of my functions though--this is a consideration I don't want to have to worry about!
Oof, Clash defaults to Active high reset, but my setup is active low, that hurt!
UART is craaaaazy.  Start/stop bits give you a (very imprecise) falling edge, we seek to the middle of the pulse, pull 8 bits, and done!  (For 8N1, it's configurable).  Allow the stop bit to run short to cut some slack if we're lagging behind the input clock.
I learn that clock constraints don't actually configure the clock, lol.  Need to go faster or learn how to introduce clock dividers.
Got LEDs lighting up according to which ASCII code I type!
Reddit finally solves my weird sync issue with UART, the USB chip has predefined baud rates it can use.
Also makes it clear that if I want to go _FAST_ for IO then I have a lot of work to do.  We'll stick with UART then!
Transmit now works (much easier, we just stay steady, no re-syncing), we have echo!
After all this work, I now have the dumbest hardware peripheral, lol.
Despite having quite a bit of error between my baud rates, I'm able to flawlessly echo day 1s input multiple times, yay!
UART really changes my model, so need to re-write a bit.
Definitely a bit trick to manage all the different clock timings.  Ideally this would really have a UART clock domain and a processing clock domain communicating over FIFOs, but that's still new territory for me!
Oof, UART (especially with required sampling, min of 3 cycles) makes my waves extremely hard to read (and generate)!  At some point I need to set it to just simulate the outside.
Doesn't seem possible to get my design up to 100MHz, bottleneck is _single_ step of the double dabble, so would be extreme tough to break it down further (and I'd be on my own).  This means I need a clock division!
Realizing I need to incorporate Lattice IP (embedded Verilog) in order to do the clock divisions, this is definitely confusing.  Luckily, someone has done this work already for Clash!
Definitely tricky to configure the second clock, but seem to have it.
It works!
Using stdin/stdout with picocom is a bit of a hack, because it can't really know when things are or aren't done.  I either need my own program to manage it or I need to adopt something like ZMODEM for a standard way over serial to send files back and forth.
Of course a well thought out UART package already exists!
