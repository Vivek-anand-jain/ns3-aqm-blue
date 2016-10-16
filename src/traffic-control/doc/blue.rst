.. include:: replace.txt
.. highlight:: cpp

BLUE queue disc
----------------

This chapter describes the BLUE [Feng99]_ queue disc implementation in |ns3|. 

BLUE is a queuing discipline that aims to
solve the bufferbloat [Buf14]_ problem. The model in ns-3 is a port of Wu-chang 
Feng's ns-1.1 BLUE model.


Model Description
*****************

The source code for the BLUE model is located in the directory ``src/traffic-control/model``
and consists of 2 files `blue-queue-disc.h` and `blue-queue-disc.cc` defining a BlueQueueDisc
class. The code was ported to |ns3| by Mohit P. Tahiliani, Vivek Jain and Sandeep Singh
based on ns-1.1 code implemented by Wu-chang Feng.  

* class :cpp:class:`BlueQueueDisc`: This class implements the main Blue algorithm:

  * ``BlueQueueDisc::DoEnqueue ()``: This routine checks whether the queue is full, and if so, drops the packets and records the number of drops due to queue overflow and calls routine ``BlueQueueDisc::IncrementPmark()``. If queue is not full, this routine calls ``BlueQueueDisc::DropEarly()``, and depending on the value returned, the incoming packet is either enqueued or dropped.

  * ``BlueQueueDisc::DropEarly ()``: The decision to enqueue or drop the packet is taken by invoking this routine, which returns a boolean value; false indicates enqueue and true indicates drop.

  * ``BlueQueueDisc::IncrementPmark ()``: This routine is called when there is heavy congestion in queue and increases the drop probability.

  * ``BlueQueueDisc::DecrementPmark ()``: This routine is called when link is idle for certain time and decrements the drop probability.

  * ``BlueQueueDisc::DoDequeue ()``: This routine calculates the average departure rate which is required for updating the drop probability in ``BlueQueueDisc::CalculateP ()``  

References
==========

.. [Feng99] W. Feng, D. Kandlur, D. Saha, K. Shin (1999, April). The Blue Queue Management Algorithms, IEEE/ACM Transactions on Networking, Vol. 10, No. 4.  Available online at `<http://www.thefengs.com/wuchang/blue/ToN-02.pdf>`_.

.. [Buf14] Bufferbloat.net.  Available online at `<http://www.bufferbloat.net/>`_.

Attributes
==========

The key attributes that the BlueQueue class holds include the following: 

* ``Mode:`` BLUE operating mode (BYTES or PACKETS). The default mode is PACKETS. 
* ``QueueLimit:`` The maximum number of bytes or packets the queue can hold. The default value is 25 bytes / packets.
* ``MeanPktSize:`` Mean packet size in bytes. The default value is 1000 bytes.
* ``Increment:`` Amount of value used to increase drop probability. The default value is 0.0025.
* ``Decrement:`` Amount of value used to decrease drop probability. The default value is 0.00025.
* ``IHoldTime:`` Minimum Time to wait before incrementing drop probability. The default value is 100 ms. 
* ``DHoldTime:`` Minimum Time to wait before decrementing drop probability. The default value is 100 ms. 
* ``IFreezeTime:`` Last time at which drop probability is incremented.
* ``DFreezeTime:`` Last time at which drop probability is decremented.
* ``DAlgorithm:`` Denotes which algorithm to use for decrementing drop probability.
* ``IAlgorithm:`` Denotes which algorithm to use for incrementing drop probability.
* ``PMark:`` Value of drop probability.

Examples
========

The example for BLUE is `blue-example.cc` located in ``src/traffic-control/examples``.  To run the file (the first invocation below shows the available
command-line options):

:: 

   $ ./waf --run "blue-example --PrintHelp"
   $ ./waf --run "blue-example --writePcap=1" 

The expected output from the previous commands are 10 .pcap files.

Validation
**********

The BLUE model is tested using :cpp:class:`BlueQueueDiscTestSuite` class defined in `src/traffic-control/test/Blue-queue-test-suite.cc`. The suite includes 2 test cases:

* Test 1: The first test checks the enqueue/dequeue with no drops and makes sure that BLUE attributes can be set correctly.
* Test 2: The second test checks the enqueue/dequeue with drops according to BLUE algorithm

The test suite can be run using the following commands: 

::

  $ ./waf configure --enable-examples --enable-tests
  $ ./waf build
  $ ./test.py -s blue-queue-disc

or  

::

  $ NS_LOG="BlueQueueDisc" ./waf --run "test-runner --suite=blue-queue-disc"
