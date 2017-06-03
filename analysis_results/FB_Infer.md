## FB Infer Analysis Results

FB Infer consists of five checkers for detecting memory safety issues in C/C++/Java/Objective-C programs: null dereference checker, memory leak checker, use-after-free checker, resource leak checker, and empty vector access checker.  We ran all the checkers on Drake.  The checkers are fully automatic as memory safety is a generic property independent of the analyzed program’s functionality.

The number of reports produced by each checker is as follows:

<table>
<tr><td>Null Dereference</td><td>420</td></tr>
<tr><td>Buffer Overrun</td><td>52</td></tr>
<tr><td>Memory Leak</td><td>74</td></tr>
<tr><td>Use After Free</td><td>7</td></tr>
<tr><td>Resource Leak</td><td>5</td></tr>
<tr><td>Empty Vector Access</td><td>1</td></tr>
</table>

### Null Dereference Checker

After manual inspection, 138 out of 420 null dereference alarms turned out to be viable. The checker detected a feasible null dereference as well as missing null checks after allocating memory with malloc, strdup, getenv, and fopen. 

For brevity, we describe 5 representative alarms as follows.


#### Alarm 1: [drake/externals/ipopt/Ipopt/src/LinAlg/IpExpansionMatrix.cpp:371](https://github.com/RobotLocomotion/ipopt-mirror/blob/aecf5abd3913eebf1b99167c0edd4e65a6b414bc/Ipopt/src/LinAlg/IpExpansionMatrix.cpp#L371)

The pointer `compressed_pos_` is allocated memory only if `NRows() > 0` (at line 361 and 362). If `NCols() > 0` and `NRows() <= 0`, then it could be null and is dereferenced at line 371.

```c
ExpansionMatrixSpace(...) : expanded_pos_(NULL), compressed_pos_(NULL)  {

358:    if (NCols()>0) {
359:       expanded_pos_  = new Index[NCols()];
360:    }
361:    if (NRows()>0) {
362:       compressed_pos_ = new Index[NRows()];
363:    }
        ...
367:    for (Index i=0; i<NCols(); i++) {      
           ...
370:       expanded_pos_[i] = ExpPos[i]-offset;
371:       compressed_pos_[ExpPos[i]-offset] = i;
        }
}
```

#### Alarm 2: [drake/externals/libbot/bot2-lcm-utils/src/tunnel/ldpc/ldpc_scheme.cpp:306](https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-lcm-utils/src/tunnel/ldpc/ldpc_scheme.cpp#L306)

The `ESIofSymbols` last assigned on line 284: 

```c
284: *ESIofSymbols = (int*) calloc(m_nbSymbolsPerPkt, sizeof(int));
```

could be null and is dereferenced at line 306, column 5 `(*ESIofSymbols)`.

#### Alarm 3: [drake/externals/lcm/lcmgen/getopt.c:51](https://github.com/lcm-proj/lcm/blob/c0a0093a950fc83e12e8d5918a0319b590356e7e/lcmgen/getopt.c#L51)

The `arg` last assigned on line 50 could be null and is dereferenced by call to `strstr()` at line 51. 

```c
50: char *arg = strdup(argv[i]);
51: char *eq = strstr(arg, "=");
```

#### Alarm 4: [drake/externals/libbot/bot2-procman/src/deputy/procman.c:101](https://github.com/RobotLocomotion/libbot2/blob/495ae366d5e380b58254368217fc5c798e72aadd/bot2-procman/src/deputy/procman.c#L101)

The `path` last assigned on line 100 could be null and is dereferenced by call to `strlen()` at line 101:

```c
100: char *path = getenv ("PATH");
101: int newpathlen = strlen (path) + strlen(params->bin_path) + 2;
```

#### Alarm 5: [drake/externals/ipopt/Ipopt/src/Algorithm/LinearSolvers/IpPardisoSolverInterface.cpp:639](https://github.com/RobotLocomotion/ipopt-mirror/blob/aecf5abd3913eebf1b99167c0edd4e65a6b414bc/Ipopt/src/Algorithm/LinearSolvers/IpPardisoSolverInterface.cpp#L639)

The pointer `mat_file` last assigned on line 637 could be null and is dereferenced by call to `fprintf()` at line 639:

```c
637: mat_file = fopen (mat_name, "w");
639: fprintf (mat_file, "%d\n", N);
```

### Buffer Overflow Checker

After manual inspection, XX out of 52 buffer-overflow alarms turned out to be viable.

### Memory Leak, Use After Free, Resource Leak, and Empty Vector Access Checkers

After manual inspection, all of 87 alarms turned out to be false. 

