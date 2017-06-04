## An Automated Analysis of Safety and Security in the Drake System

This document presents the results of running Petablox and other state-of-the-art automated static analysis tools on the Drake codebase to uncover safety and security issues.  We present analytics results for various Drake modules, describe potential issues we discovered, provide pointers to raw output of the tools, and outline future directions to improve the tools.

### The Checkers

We ran various checkers in four different automated static analysis tools on the Drake codebase obtained from http://drake.mit.edu/from_source.html

1. Petablox: API misuse checkers (http://www.seas.upenn.edu/~mhnaik/pubs/sec16.pdf)
2. FB Infer: Memory safety checkers (http://FBinfer.com)
3. MIT Kint: Integer overflow checker (http://css.csail.mit.edu/kint)
4. Coverity: bug pattern checkers (https://scan.coverity.com)

All of the above tools except Coverity are freely available.  Coverity provides a free service for github hosted projects (such as Drake); its results of analyzing Drake will be available in 48 hours.

### Key Findings

<ul>
<li>Petablox and FB Infer were able to run successfully on XXX thousand lines of source code in the Drake codebase.  This attests to the suitability of these tools for analyzing large and complex autonomous software systems.</li>

<li>FB Infer found 138 potential null dereference bugs and Petablox found 2 buffer underruns and 9 null dereferences. Among them, we describe three representative alarms as follows. 
<table>
  <tr> 
    <td> Link to details </td> 
    <td> High-level description </td> 
  </tr>

  <tr> 
    <td> <a href="analysis_results/FB_Infer.md#alarm-1-drakeexternalsipoptipoptsrclinalgipexpansionmatrixcpp371">ND in externals/ipopt</a> </td> 
    <td> A pointer is allocated memory depending on a condition. But the condition may not hold when the pointer is accessed. </td> 
  </tr>

  <tr> 
    <td> <a href="analysis_results/Petablox.md#alarm-1-missing-non-null-check-drakemultibodyparsersurdf_parsercc1320">ND in drake/multibody</a> </td> 
    <td> A xml attribute value is retrieved and used. If the attribute does not exist, null dereference can occur. By testing, we confirmed it causes a crash. </td> 
  </tr>
  
  <tr> 
    <td> <a href="analysis_results/Petablox.md#alarm-2-missing--1-check-externalsipoptthirdpartymetismetis-40libsfmc352">BU in externals/ipopt</a> </td> 
    <td> An integer element is retrieved from a priority queue and used as index of a buffer. If the queue is empty, the index can be -1.</td> 
  </tr>
    
</table>
</li>

<li>Future work in suppressing false alarms by synthesizing rules</li>
</ul>

### Checker Analytics

<table>
  <tr> 
    <td> Modules </td> 
    <td> Petablox </td> 
    <td> FBInfer </td> 
    <td> Kint </td> 
    <td> Coverity </td>
  </tr>

  <tr> 
    <td>  drake/automotive </td> 
    <td>  0 </td> 
    <td>  22 </td> 
    <td>  </td> 
    <td>  </td> 
  </tr>




  <tr> 
    <td> drake/multibody </td> 
    <td> 559 </td> 
    <td> 1 </td> 
    <td>  </td> 
    <td>  </td> 
  </tr>

  <tr> 
    <td> drake/systems </td> 
    <td> 1203 </td> 
    <td> 13 </td> 
    <td>  </td>
    <td>  </td> 
  </tr>

  <tr> 
    <td> drake/common </td> 
    <td> 1008 </td> 
    <td> 2 </td> 
    <td>  </td> 
    <td>  </td> 
  </tr>

  <tr> 
    <td> externals/libbot </td> 
    <td> 38 </td> 
    <td> 108 </td> 
    <td>  </td> 
    <td>  </td> 
  </tr>

  <tr> 
    <td> externals/ipopt </td> 
    <td> 13 </td> 
    <td> 70 </td> 
    <td>  </td> 
    <td>  </td> 
  </tr>

  <tr> 
    <td> others </td> 
    <td>  </td> 
    <td>  </td> 
    <td>  </td>
    <td>  </td> 
  </tr>

</table>


### Analysis Results

Our analysis of potential safety/security issues discovered in Drake by the tools is available in files under directory [analysis_results](analysis_results).

### Detailed Output

The raw output of running the tools on Drake is available in folders under directory [raw_logs](raw_logs).

