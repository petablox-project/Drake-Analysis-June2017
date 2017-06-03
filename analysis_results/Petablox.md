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

The return value checker semantically analyzes return value checks across different calls to each function and reports unusual ones as potential bugs.  For instance, if at least 70% of all calls to the malloc() function check that the return value is non-null, then any of the remaining calls to the malloc() function that do not check for non-nullness are reported.

<table>
  <tr>
  	<td>
  		Location
  	</td>
  	<td>
  		Description
  	</td>
  </tr>

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
  		externals/libbot/bot2-lcm-utils/src/tunnel/introspect.c:107
  	</td>
  	<td>
  		New function should check if subscription fails and do something if it does.
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-frames/src/renderer/frame_modifier_renderer.c:132
  	</td>
  	<td>
  		File may not exist. However we create the string that represents the file name, so we may have a guarantee that the file exists.
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/sfm.c:352
  	</td>
  	<td>
  		Queue might be empty and should be checked for -1.
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/ipopt/ThirdParty/Metis/metis-4.0/Lib/sfm.c:115
  	</td>
  	<td>
  		same as above
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-lcmgl/src/bot_lcmgl_render/lcmgl_bot_renderer.c:122
  	</td>
  	<td>
  		the hash table lookup needs to be checked to see if the key is in the table
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-vis/src/bot_vis/param_widget.c:166
  	</td>
  	<td>
  		same as above
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-lcmgl/src/bot_lcmgl_render/lcmgl_bot_renderer.c:46
  	</td>
  	<td>
  		same as above
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-vis/src/bot_vis/param_widget.c:76
  	</td>
  	<td>
  		This is a bug for the same reason as above, depending on whether GNOME does null checks on things passed to its APIs
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-procman/src/deputy/procman.c:100
  	</td>
  	<td>
  		Result of getenv should be checked for NULL 
  	</td>
  </tr>

  <tr>
  	<td>
  		externals/libbot/bot2-vis/src/bot_vis/viewer.c:352
  	</td>
  	<td>
  		Result of write not checked for success (so preferences may not be successfully saved)
  	</td>
  </tr>

</table>