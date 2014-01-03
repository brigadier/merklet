# Merklet #

Reimplementation of Riak's old merkle tree module, but in a more readable
manner (according to me). Other difference include:

- Renaming some functions just because
- Insertion wants only binaries for keys and values, but makes no assumption
  with regards to hashing.
- Deleting nodes doesn't leave empty inner nodes as part of the tree, and
  inner nodes with a single child node see the child promoted to the current
  level.
- Slightly more efficient diffing. This is due to not having as many leftover
  inner nodes, and using difference lists on offsets of two inner nodes instead
  of iterating across all offsets. I.e. the cost is linear with the number of
  children of an inner node, rather than a flat minimal 255 rounds for each.
- The diff algorithm explicitly points out differences in keys *and* values,
  not just keys.

Further documentation (and changes) to come.

Todo list:

- I'd like to add a serialization function that would be more or less
  language-agnostic.
- I'd like to add a functionality that is more protocol-like and would
  make it possible to do incremental tree diffing over the network, as
  opposed to the current one, which requires both trees to fully be on the
  node.

## Compiling ##

    $ rebar compile

## Running Tests

    $ rebar get-deps compile --config test.rebar.config
    $ rebar eunit --config test.rebar.config skip_deps=true

## Usage

```
1> T4 = merklet:insert({term_to_binary("wf"), <<"hehe">>}, T3 = merklet:insert({term_to_binary("c"), <<"third">>}, T2 = merklet:insert({<<"b">>, <<"other">>}, T1 = merklet:insert({<<"a">>, <<"val">>}, T0 = undefined)))).
#inner{hashchildren = <<106,248,148,67,122,133,161,251,
                        225,195,114,6,143,172,176,84,172,
                        184,27,93>>,
       children = [{134,
                    #leaf{userkey = <<"a">>,
                          hashkey = <<134,247,228,55,250,165,167,252,225,93,29,220,
                                      185,234,234,234,55,118,103,184>>,
                          hashval = <<57,246,156,39,143,70,22,84,71,243,13,16,172,
                                      245,66,119,170,163,213,...>>,
                          hash = <<65,36,144,16,123,227,33,235,119,175,10,124,246,
                                   206,104,227,161,8,...>>}},
                   {211,
                    #inner{hashchildren = <<170,222,15,48,84,153,7,5,86,207,
                                            227,247,171,218,50,2,120,249,218,
                                            245>>,
                           children = [{29,
                                        #leaf{userkey = <<131,107,0,1,99>>,
                                              hashkey = <<211,29,241,167,251,188,205,162,69,235,0,101,
                                                          90,...>>,
                                              hashval = <<52,251,51,0,185,167,123,235,220,152,142,195,
                                                          ...>>,
                                              hash = <<234,174,116,145,240,25,48,115,149,166,49,...>>}},
                                       {191,
                                        #leaf{userkey = <<131,107,0,2,119,102>>,
                                              hashkey = <<211,191,171,22,1,47,67,156,204,229,77,113,...>>,
                                              hashval = <<66,82,91,182,211,176,220,6,187,120,174,...>>,
                                              hash = <<139,130,66,190,81,141,104,185,159,140,...>>}}],
                           offset = 1}},
                   {233,
                    #leaf{userkey = <<"b">>,
                          hashkey = <<233,215,31,94,231,201,45,109,201,233,47,253,
                                      173,23,184,189,73,65,...>>,
                          hashval = <<208,148,30,104,218,143,56,21,31,248,106,97,
                                      252,89,247,197,207,...>>,
                          hash = <<30,97,58,207,252,237,241,68,194,28,170,220,89,
                                   211,245,221,...>>}}],
       offset = 0}
2> merklet:delete(term_to_binary("wf"), T4).
#inner{hashchildren = <<6,190,175,250,248,245,80,92,143,
                        119,250,104,201,226,231,127,189,
                        10,11,21>>,
       children = [{134,
                    #leaf{userkey = <<"a">>,
                          hashkey = <<134,247,228,55,250,165,167,252,225,93,29,220,
                                      185,234,234,234,55,118,103,184>>,
                          hashval = <<57,246,156,39,143,70,22,84,71,243,13,16,172,
                                      245,66,119,170,163,213,...>>,
                          hash = <<65,36,144,16,123,227,33,235,119,175,10,124,246,
                                   206,104,227,161,8,...>>}},
                   {211,
                    #leaf{userkey = <<131,107,0,1,99>>,
                          hashkey = <<211,29,241,167,251,188,205,162,69,235,0,101,
                                      90,83,57,132,52,158,32,...>>,
                          hashval = <<52,251,51,0,185,167,123,235,220,152,142,195,
                                      237,208,212,166,164,42,...>>,
                          hash = <<234,174,116,145,240,25,48,115,149,166,49,44,
                                   177,64,179,22,57,...>>}},
                   {233,
                    #leaf{userkey = <<"b">>,
                          hashkey = <<233,215,31,94,231,201,45,109,201,233,47,253,
                                      173,23,184,189,73,65,...>>,
                          hashval = <<208,148,30,104,218,143,56,21,31,248,106,97,
                                      252,89,247,197,207,...>>,
                          hash = <<30,97,58,207,252,237,241,68,194,28,170,220,89,
                                   211,245,221,...>>}}],
       offset = 0}
3> merklet:keys(T3).
[<<"a">>,<<"b">>,<<131,107,0,1,99>>]
4> merklet:keys(T4).
[<<"a">>,<<"b">>,<<131,107,0,1,99>>,<<131,107,0,2,119,102>>]
5> merklet:diff(T3,T4).
[<<131,107,0,2,119,102>>]
6> merklet:diff(T3,undefined).
[<<"a">>,<<"b">>,<<131,107,0,1,99>>
```

## Notes ##

The implementation is still incomplete and should not be used while this notice
is here.

More docs to come.