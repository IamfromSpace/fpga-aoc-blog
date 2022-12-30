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
This UART package is pretty sweet, it does a bunch of type math to ensure the baud rate is correct, neat!  Sort of wishing the PLL package did the same, because that's not actually checked.
I'm inspired to learn some type math, I get my delay/vector queue (a lazy stand-in for learning how FIFOs work) to be fully generic, using the minimum sized registers to count up delay and indexes regardless of the number of clock cycles or elements, neat!
Day 2 goes swimmingly!  Since I have the device this time, I don't know the answer until _it_ answers it.  The first time around I used the wavedrom rendering, because I was still pretty far off from actual synthesis and interfacing (I didn't even have the board yet!).
Day 3.... oh, shit I need RAM for sorting!
I spend a lot of time looking up and thinking about sorting.  A sorting network via bitonic sort seems very interesting, but it still seems too large for 64 values.
Nope, I'm dumb, I don't need to sort anything! I can just do a simple histogram
Oh, but I do absolutely need RAM.  My implementation sort of "outsmarts" itself.  A huge problem is my usage of a dynamic rotation of a Vec 48.  This costs literally 50k LCs, which isn't even close to an option.  Even if it was a fixed rotate, I'm still at about 10k LCs based on trying to push through my hist counting fast enough.  I have just three clock cycles to work with, and if I try and do it all in one step, I essentially have a gigantic mux from hell.  If I try to do it in 24 steps, then I have an enormous number of registers in use.  My attempt at a middle ground didn't work.
In the end a "double FIFO" works out nicely.  They work on the same clock which simplifies things, and as long as it's big enough we never over run, so our writer never actually checks before writing.  I try for some time to not pass a pointer from the writer to the reader for some reason, thinking I can outsmart the way you do these things, and then finally realize that the normal approach to this is just dramatically simpler.
Ouch, I finally get it working but I get burned by different truncation semantics between the simulation and the actual hardware.  It _looks_ like I've screwed up my RAM (partly since that's what's new and I expect i screwed up), but the final hint comes when I hard code the value being written into RAM and my max clock speed _slows down_.  That seems weird, right?  The reason is that because the value being written was undefined, there was no reason to actually do any logic with it!  So a bunch of things got optimized away to more efficiently do the wrong thing.
