# DASH-NS3
A simulation model for HTTP-based adaptive streaming applications

If you use the model, please reference "Simulation Framework for HTTP-Based Adaptive Streaming Applications" by Harald Ott, Konstantin Miller, and Adam Wolisz, 2017

## NEEDED FILES
Just drop the repository into the contrib/ folder of ns-3 (only works with ns version between 3.27 and 3.30)

Since I've already received a lot of questions about errors that arise from this mistake:

![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) !! DO NOT RENAME THE FOLDER !! ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) 

Its name needs to remain 'dash'

## PROGRAM EXECUTION
The following parameters have to be specified for program execution:
- simulationId: The Id of this simulation, to distinguish it from others, with same algorithm and number of clients, for logging purposes.
- numberOfClients: The number of streaming clients used for this simulation.
- segmentDuration: The duration of a segment in microseconds.
- adaptationAlgo: The name of the adaptation algorithm the client uses for the simulation. The 'pre-installed' algorithms are tobasco, festive and panda.
- segmentSizeFile: The relative path (from the ns-3.x/ folder) of the file containing the sizes of the segments of the video. The segment sizes have to be provided as a (n, m) matrix, with n being the number of representation levels and m being the total number of segments. A two-segment long, three representations containing segment size file would look like the following:

 1564 22394  
 1627 46529  
 1987 121606  

One possible execution of the program would be:
```bash
./waf --run="tcp-stream --simulationId=1 --numberOfClients=3 --adaptationAlgo=panda --segmentDuration=2000000 --segmentSizeFile=contrib/dash/segmentSizes.txt"
```


## ADDING NEW ADAPTATION ALGORITHMS
The adaptation algorithm base class is located in src/applications/model/adaptation-algorithm/. If it is desired to implement a new adaptation algorithm, a separate source and header file for the algorithm can be created in the adaptation-algorithm/ folder. An example of how a header file looks like can be seen here:

```c++
#ifndef NEW_ALGORITHM_H
#define NEW_ALGORITHM_H

#include "tcp-stream-adaptation-algorithm.h"

namespace ns3 {
/**
 * \ingroup tcpStream
 * \brief Implementation of a new adaptation algorithm
 */
class NewAdaptationAlgorithm : public AdaptationAlgorithm
{
public:

NewAdaptationAlgorithm ( const videoData &videoData,
                         const playbackData & playbackData,
			 const bufferData & bufferData,
			 const throughputData & throughput );

algorithmReply GetNextRep ( const int64_t segmentCounter );
};
} // namespace ns3
#endif /* NEW_ALGORITHM_H */
```

An adaptation algorithm must return a data structure 'algorithmReply' containing the following members:

```c++
int64_t nextRepIndex; // representation level index of the next segement to be downloaded by the client
int64_t nextDownloadDelay; // delay time in microseconds when the next segment shall be requested from the server
int64_t decisionTime; // time in microsends when the adaptation algorithm decided which segment to download next, only for logging purposes
int64_t decisionCase; // indicate in which part of the adaptation algorithm's code the decision was made, which representation level to request next, only for logging purposes
int64_t delayDecisionCase; // indicate in which part of the adaptation algorithm's code the decision was made, how much time in microsends to wait until the segment shall be requested from server, only for logging purposes
```

Next, it is necessary to include the following lines to the top of the source file.

```c++
NS_LOG_COMPONENT_DEFINE ("NewAdaptationAlgorithm");
NS_OBJECT_ENSURE_REGISTERED (NewAdaptationAlgorithm);
```

It is obligatory to inherit from AdaptationAlgorithm and implement the algorithmReply GetNextRep ( const int64_t segmentCounter ) function. Then, the header and source files need to be added to src/applications/wscript. Open wscript and add the files with their path, just like the other algorithm files have been added. Additionally, it is necessary to add the name of the algorithm to the if-else-if block in the TcpStreamClient::Initialise (std::string algorithm) function, just like the other implemented algorithms have been added, see the following code taken from tcp-stream-client.cc:

```c++
if (algorithm == "tobasco")
  {
    algo = new TobascoAlgorithm (m_videoData, m_playbackData, m_bufferData, m_throughput);
  }
else if (algorithm == "panda")
  {
    algo = new PandaAlgorithm (m_videoData, m_playbackData, m_bufferData, m_throughput);
  }
else if (algorithm == "festive")
  {
    algo = new FestiveAlgorithm (m_videoData, m_playbackData, m_bufferData, m_throughput);
  }
else
  {
    // Stop program
  }
```
Lastly, the header file of the newly implemented adaptation algorithm needs to be included in the TcpStreamClient header file.

The resulting logfiles will be written to mylogs/algorithmName/numberOfClients/
