# toml_c.c3l
[toml-c](https://github.com/arp242/toml-c) binding for C3.

# Usage
```c3
module toml_example;

import std::io;
import std::os;
import toml_c;

alias tc = module toml_c;

ZString doc = `
ints  = [1, 2, 3]
mixed = [1, 'one', 1.2]

[[aot]]
k = 'one'
[[aot]]
k = 'two'
`;

fn int main(String[] args)
{
  CChar[200] errbuf;
  tc::Table *tbl = tc::parse(&doc,&errbuf,errbuf.len);

  if(!tbl)
  {
    io::eprintfn("ERROR: %s",(ZString)&errbuf);
    os::exit(1);
  }

  defer tbl.free();

  // Array of ints:
  Array *arr = tbl.array("ints");
  CInt l = tbl.len();

  io::printn("ints:");
  for(CInt i = 0; i < l; i++)
  {
    io::printfn("  index %d = %d\n",i,arr.int_value(i).u.i);
  }
  io::printn("");

  // Mixed array:
  arr = tbl.array("mixed");
  l = arr.len();

  io::printn("mixed:");
  for(int i = 0; i < l; i++)
  {
    tc::Value v = arr.int_value(i);
    if(v.ok)
    {
      io::printfn("  index %d = %d",i,v.u.i);
      continue;
    }
    v = arr.double_value(i);
    if(v.ok)
    {
      io::printfn("  index %d = %0.17f",i,v.u.d);
      continue;
    }
    v = arr.string(i);
    if(v.ok)
    {
      io::printfn("  index %d = %s",i,v.u.s);
      continue;      
    }
  }
  io::printn("");
  
  // Array-of-tables works just like inline tables.
  arr = tbl.array("aot");
  l = arr.len();

  for(int i = 0; i < l; i++)
  {
    Table *t = arr.table(i);
    Value v = t.string("k");
    io::printfn("aot[%d].k = \"%s\"",i,v.u.s);
  }
  return 0;
}
```

