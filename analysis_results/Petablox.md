## Petablox Analysis Results

Petablox consists of six checkers for detecting API misuses in C/C++ programs: return value checker, argument checker, causality checker, condition checker, integer overflow checker and format string checker.  We ran all checkers except for the argument checker on Drake.  The checkers are fully automatic: they do not require specifying correct API usages, or even which functions constitute an API. They automatically mine semantic patterns of API usage across a large codebase like Drake, perform statistical analysis of these patterns, and report infrequent patterns as potential API misuses.

The number of reports produced by each checker is as follows:
<table>
  <tr>
    <td> return value checker </td>
    <td> 71 </td>    
  </tr>

  <tr>
    <td> causality checker </td>
    <td> 248 </td>    
  </tr>

  <tr>
    <td> condition checker </td>
    <td> 3237 </td>    
  </tr>

  <tr>
    <td> integer overflow checker </td>
    <td> 3 </td>    
  </tr>

  <tr>
    <td> format string checker </td>
    <td> 23 </td>    
  </tr>

</table>

### Return Value Checker

The return value checker semantically analyzes return value checks across different calls to each function and reports unusual ones as potential bugs.  For instance, if at least 70% of all calls to the malloc() function check that the return value is non-null, then any of the remaining calls to the malloc() function that do not check for non-nullness are reported.  It reports 71 potential missing checks with 80% as the threshold of majority patterns. We manually went over them and found 11 of them are very likely bugs. 

#### Alarm 1: `missing non-null check` [drake/multibody/parsers/urdf_parser.cc:1320](https://github.com/RobotLocomotion/drake/blob/master/drake/multibody/parsers/urdf_parser.cc#L1320)

```c++
// drake/thirdParty/zlib/tinyxml2/tinyxml2.h
class TINYXML2_LIB XMLElement : public XMLNode
{
  ...
  const char* Attribute( const char* name, const char* value=0 ) const;
  ...
}

//drake/multibody/parsers/urdf_parser.cc
std::shared_ptr<RigidBodyFrame<double>> MakeRigidBodyFrameFromUrdfNode(..., 
  const tinyxml2::XMLElement& link, ...)
{
  ...
1320:  string body_name = link.Attribute("link");
  ...
}
```

In line-1320, `link.Attribute("link")` returns a `const char*`, which may be null. We tested the following piece of code, which crashed with `Segmentation fault: 11` error. This is because the string constructor cannot initialize with null pointer properly. Thus, the multibody module of Drake could be crashed by line-1320 due to segmentation fault. A non-null check of return value of `link.Attribute("link")` is necessary before it is used to initialize the string `body_name`.

```c++
#include <string>
int main()
{
     const char* value = nullptr;
     std::string t = value;
     return 0;
}
```

#### Alarm 2: `missing -1 check` [externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/sfm.c:352](https://github.com/RobotLocomotion/ipopt-mirror/blob/aecf5abd3913eebf1b99167c0edd4e65a6b414bc/ThirdParty/Metis/metis-4.0/Lib/sfm.c#L352)

```c++
// drake/externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/pqueue.c
int PQueueGetMax(PQueueType *queue)
{
  ...
   if (queue->nnodes == 0)
     return -1;
  ...
}

// drake/externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/sfm.c
void FM_2WayNodeRefine2(CtrlType *ctrl, GraphType *graph, float ubfactor, int npasses)
{
  ...
352: higain = PQueueGetMax(&parts[to]);
353: if (moved[higain] == -1) { ... }
  ...
358: pwgts[2] -= (vwgt[higain]-rinfo[higain].edegrees[other]);
  ...
}
```

In line-352, the index variable `higain` is assigned by the return value of `PQueueGetMax()`, which may return `-1`.
Thus, the access to arrays like `moved`, `vwgt` and `rinfo` used in line-353 and line-358 may get segmentation fault, 
or even worse, the code continues execution silently in an unexpected way. 


The remaining reports are buggy due to similar reasons, and we briefly summarize them as follows.
<table>
  <tr>
  	<td>
  		Location
  	</td>
  	<td>
  		Description
  	</td>
  </tr>

  <!--
  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/drake/blob/master/drake/multibody/parsers/urdf_parser.cc#L1320"> drake/multibody/parsers/urdf_parser.cc:1320 </a>
  	</td>
  	<td>
  		Should do the null check after return; this would be a bug if the string constructor cannot initialize with null pointer properly.
  	</td>
  </tr>

  <tr>
    <td>
      <a href="https://github.com/RobotLocomotion/ipopt-mirror/blob/aecf5abd3913eebf1b99167c0edd4e65a6b414bc/ThirdParty/Metis/metis-4.0/Lib/sfm.c#L352">externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/sfm.c:352 </a>
    </td>
    <td>
      Queue might be empty and should be checked for -1.
    </td>
  </tr>
  -->

  <tr>
    <td>
      <a href="https://github.com/RobotLocomotion/ipopt-mirror/blob/aecf5abd3913eebf1b99167c0edd4e65a6b414bc/ThirdParty/Metis/metis-4.0/Lib/sfm.c#L115">externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/sfm.c:115 </a>
    </td>
    <td>
      Queue might be empty and should be checked for -1.
    </td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-lcm-utils/src/tunnel/introspect.c#L107">externals/libbot/bot2-lcm-utils/src/tunnel/introspect.c:107</a>
  	</td>
  	<td>
  		New function should check if subscription fails and do something if it does.
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-frames/src/renderer/frame_modifier_renderer.c#L132">externals/libbot/bot2-frames/src/renderer/frame_modifier_renderer.c:132 </a>
  	</td>
  	<td>
  		File may not exist. However we create the string that represents the file name, so we may have a guarantee that the file exists.
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-lcmgl/src/bot_lcmgl_render/lcmgl_bot_renderer.c#L122">externals/libbot/bot2-lcmgl/src/bot_lcmgl_render/lcmgl_bot_renderer.c:122
  	</td>
  	<td>
  		the hash table lookup needs to be checked to see if the key is in the table
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-vis/src/bot_vis/param_widget.c#L166">externals/libbot/bot2-vis/src/bot_vis/param_widget.c:166 </a>
  	</td>
  	<td>
  		same as above
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-lcmgl/src/bot_lcmgl_render/lcmgl_bot_renderer.c#L46">externals/libbot/bot2-lcmgl/src/bot_lcmgl_render/lcmgl_bot_renderer.c:46 </a>
  	</td>
  	<td>
  		same as above
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-vis/src/bot_vis/param_widget.c#L76" > externals/libbot/bot2-vis/src/bot_vis/param_widget.c:76 </a>
  	</td>
  	<td>
  		This is a bug for the same reason as above, depending on whether GNOME does null checks on things passed to its APIs
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-procman/src/deputy/procman.c#L100" > externals/libbot/bot2-procman/src/deputy/procman.c:100 </a>
  	</td>
  	<td>
  		Result of getenv should be checked for NULL 
  	</td>
  </tr>

  <tr>
  	<td>
  		<a href="https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-vis/src/bot_vis/viewer.c#L352" > externals/libbot/bot2-vis/src/bot_vis/viewer.c:352 </a>
  	</td>
  	<td>
  		Result of write not checked for success (so preferences may not be successfully saved)
  	</td>
  </tr>

</table>