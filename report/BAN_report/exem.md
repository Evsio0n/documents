Introduce
=========

[ RS485 feature]{}

-   485 is Fully balanced

-   485 handles multiple drivers and receivers

-   Better common-mode noise rejection (-7 to +12Volts)

-   Sensitivity of 200mV in receivers

-   Drivers give up to 5 volts balanced output

-   Can stand contention, driver shuts down by itself

-   High input resistance (12K ohms)

-   Hysteresis of 50 mv to overcome diff. noise

![485NET []{data-label="Overview"}](img/485net){width="5cm"}

[The Principle of The BAN’s Clock Synchronization]{}

-   In order to synchronize the device clock, it is necessary to add a
    clock line between Master and Slave, where Master generates
    synchronous clock square wave.

![synchronous[]{data-label="Overview"}](img/syncchart){width="6cm"}

[Our Design]{} We designed it like this：

![BAN []{data-label="Overview"}](img/overview){width="10cm"}

[BUS]{}

1.  Adding a clock line from RS485 to form the basic hardware circuit of
    BAN bus.

2.  The clock line is used to generate Millisecond-level pulses for
    clock synchronization.Later aliases:SYNC\_CLK

3.  D +, D - is differential signal .

we assumed that the device mounted on the bus has one Master and two
Slaves.

[Local Clock Counter & SYNC\_CLK]{}

-   Local Clock Counter , Each device has a local clock counter. The
    timing granularity is 100us.

-   SYNC\_CLK,The device on BAN bus uses clock line to complete clock
    synchronization. Master’s timer generates synchronization pulse,
    Slave’scaptures synchronized pulse to complete synchronization.

[Synchronization process]{}

-   a\. master and slave power on and initialize peripherals.

-   b\. master starts sending square wave signals,at The first rising edge of
    square wave signal:

    1.  The master starts Master’s Local Clock Counter(MLCC).

    2.  Slave captures this rising edge at the same time.Then start
        Slave’s Local Clock Counter(SLCC).

    3.  The delay of the rising edge of the clock line transmission can
        be neglected, So we think that MLCC and SLCC start counting at
        the same time.

-   c\. Slave calibrates SLCC at each subsequent SYNC\_CLK rising edge.

![BAN []{data-label="Overview"}](img/abstim){width="6cm"}

[Clock Difference Calibration]{}

SYNC\_CLK is produced by Master with fixed period and precise time.Let’s
take a period of 16 milliseconds for example.

MLLC and SLLC start counter at the sametime(first SYNC\_CLK rising
edge). Their timing granularity is small, 100 microseconds.

So when the system runs, there are two counts on each device, the Local
Clock Count(Every 100 microseconds plus 1) and the SYNC\_CLK Count(Every
16 milliseconds plus 160).Slave calibrates SLCC at each subsequent
SYNC\_CLK rising edge by Make those two counts the same.

[Clock Difference Calibration]{}

![BAN []{data-label="Overview"}](img/flow){width="10cm"}

[Demo implementation]{}

The following is the demo wiring and structure：

![demo[]{data-label="Overview"}](img/demo0){width="10cm"}

workflow
========

[Demo workflow]{}

1.  Master and Slave Power on。 Peripheral Driver Initialization

2.  The Master sends a synchronization command, when ensuring that the
    slave is online:

    -   a.start timer2 to Generating Pulse Square Wave for synchronous。

    -   b\. start timer3 for MLCC(Master Local Clock Counter).

3.  When Slave receives a synchronization command from Master, and Slave
    captures the first rising edge of SYNC\_CLK:

    -   a\. start timer3 for SLCC(Slave Local Clock Counter).

    -   b\. Continue to capture the rising edge.

[Demo Workflow: Synchronize details]{}

local\_tm\_t is the type of time described in Master and Slave code:

    typedef struct local_time_struct{
     unsigned char sec:7;   //second
     unsigned char min:7;   //min
     unsigned char hour:5;  //hour
     unsigned int  day;     //day
     unsigned int  jiffies; //LCC
     unsigned int  jiffies_pwm;   //SYNC_CLK counter
    } local_tm_t;

[Slave Rising Edge Capture Interrupt]{}

tick\_sync is called every 8ms to synchronize the local timestamp:

    /* every 8ms 80*100us  */
    void tick_sync(void)
    {
      if( dev.local_tm.jiffies_pwm + 80 < 10000  ){
      dev.local_tm.jiffies_pwm = dev.local_tm.jiffies_pwm  + 80;
      }
      if( dev.local_tm.jiffies_pwm + 80 >= 10000  ){
        dev.evt.sec_need_update=1;
        dev.local_tm.jiffies_pwm = 80 - (10000 - dev.local_tm.jiffies_pwm);
      }
      dev.local_tm.jiffies = dev.local_tm.jiffies_pwm;		//synchronous
    }

[Synchronize details:test point]{}

Enter this code every 100us：

    /*
     enter this code  every 100us。
    */
    void Tick(void)
    {
      dev.local_tm.jiffies ++;
      if( dev.local_tm.jiffies % 80 == 0){
       //GPIO A5 for compare.
       HAL_GPIO_WritePin(GPIOA,GPIO_PIN_5,1-HAL_GPIO_ReadPin(GPIOA,GPIO_PIN_5));
     }
      /* one second passed */
      if(dev.local_tm.jiffies >= 10000 ){
       dev.local_tm.jiffies  = 0;
     }
    }

Here the slave GPIOA5 has square wave output relative to the clock
synchronization line. It’s a test point.

TEST Data
=========

[Waveform: Unsynchronized]{}

Let:

-   The local clock inverts the GPIOA5 level state every 8ms. This test
    channel is marked CH\_LOCAL.

-   The synchronous line generates a synchronous square wave with a half
    period of 8ms from the Master. This test channel is marked CH\_SYNC.

-   In the interrupt processing function of the slave to capture the
    rising edge of the synchronization line, the clock is not
    synchronized, that is to say, the following code is commented out:

        //dev.local_tm.jiffies = dev.local_tm.jiffies_pwm;		//synchronize code
          

The following screenshots show the unsynchronized screenshots:

[Waveform: Unsynchronized]{}

Unsynchronized waveform 1\[nosyncwave1\]：

![nosync1[]{data-label="report"}](img/nosync1){width="10cm"}

[Waveform: Unsynchronized]{}

Unsynchronized waveform 2：

![nosync2[]{data-label="report"}](img/nosync2){width="10cm"}

[Waveform: Unsynchronized]{}

Unsynchronized waveform 3：

![nosync3[]{data-label="report"}](img/nosync3){width="10cm"}

[Waveform: Unsynchronized]{}

Unsynchronized waveform 4：

![nosync4[]{data-label="report"}](img/nosync4){width="10cm"}

[Some explanations of unsynchronized waveforms]{}

Return this document to unsynchronized waveform 1 ( \[nosyncwave1\]) and
turn the page with the right key of the keyboard direction key (from
unsynchronized waveform 1 to unsynchronized waveform 4). At this time,
the animation effect on the screen can restore the appearance of the
oscilloscope at that time.

Because the local clock is not synchronized, each difference
accumulates, resulting in such a dynamic effect.

[Waveform: Synchronization ]{} Let:

-   The local clock inverts the GPIOA5 level state every 8ms. This test
    channel is marked CH\_LOCAL.

-   The synchronous line generates a synchronous square wave with a half
    period of 8ms from the Master. This test channel is marked CH\_SYNC.

-   In the interrupt processing function of the slave capturing the
    rising edge of the synchronization line, the clock is synchronized,
    that is to say, the following code is active:

        dev.local_tm.jiffies = dev.local_tm.jiffies_pwm;		//synchronize code
          

The following screenshots show the screenshots in the case of
synchronization.

[Waveform: Synchronization]{}

synchronized waveform :

![report jiffies[]{data-label="report"}](img/sync){width="10cm"}

[Waveform: Synchronization]{}

Since the rising edge of each synchronization line is captured by slave
computer, synchronization interrupt will be triggered. In interrupt
processing, the local clock error of each cycle will be eliminated.

So the synchronized waveform looks fixed and the phase is relatively
consistent. But:

If we amplify the size of the oscilloscope, we will find that there are
difference. The following are some synchronization delay. The size of
the oscilloscope is already 50us. We are concerned about the
synchronization time.

[Waveform: Synchronization]{}

Synchronization delay:

![Synchronization delay1[]{data-label="report"}](img/mis1){width="10cm"}

[Synchronization delay]{}

Synchronization delay:

![Synchronization delay1[]{data-label="report"}](img/mis1){width="10cm"}

[Synchronization delay]{}

Synchronization delay:

![Synchronization delay2[]{data-label="report"}](img/mis2){width="10cm"}

[Synchronization delay]{}

Synchronization delay:

![Synchronization delay3[]{data-label="report"}](img/mis3){width="10cm"}

[Synchronization delay]{}

![delay[]{data-label="report"}](img/delay){width="7cm"}

As can be seen from the figure above, the maximum phase difference
between CH\_LOCAL and CH\_SYNC is -3.771 -（-104.3） = 100.529us when
sampling 4.155k times.

[summary]{}

Data transmission through serial ports can cause some delays (although
there is no blocking at the time of sending, receiving in the interrupt,
reading registers directly). The baudrate we used in this example is
115200\*2.

Software-only CRC verification will inevitably lead to a certain delay.
If it is necessary to verify, the driver of CRC hardware peripherals
should be realized in this project.

The error of demo synchronization has been tested many times, among
which there are transformed master-slave roles, and the maximum error is
less than 500 us. The most common is within 300us.

The test code logic is not complex, real-time processing is in the
interrupt context, and STM32F4 uses Cortex M4’s Nested Vectored
Interrupt Controller (NVIC) real-time is very high, there should be
little optimization space.
