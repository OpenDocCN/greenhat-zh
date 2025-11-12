# 附录 B. IDC/SDK 交叉引用

![无标题图片](img/httpatomoreillycomsourcenostarchimages854059.png.jpg)

下表用于将 IDC 脚本函数映射到它们的 SDK 实现。该表的目的在于帮助熟悉 IDC 的程序员了解如何使用 SDK 函数执行类似操作。这种表的需求有两个原因：（1）IDC 函数名称与其 SDK 对应项不匹配，以及（2）在某些情况下，单个 IDC 函数由几个 SDK 操作组成。该表还揭示了 SDK 利用 *netnodes* 作为将信息存储到 IDA 数据库的一种方式的一些方法。具体来说，当我们回顾 IDC 数组操作函数时，可以看出 netnodes 被用于实现 IDC 数组的方式。

该表试图使 SDK 描述简明扼要。为此，省略了错误检查代码以及许多 C++ 语法元素（特别是大括号 `{`）。许多 SDK 函数通过将数据复制到调用者提供的缓冲区中返回结果。为了简洁，这些缓冲区未声明。为了保持一致性，这些缓冲区被命名为 `buf`，其大小在大多数情况下假定为 1,024 字节，这是 IDA 6.1 SDK 的 `MAXSTR` 常量的值。最后，仅在变量的使用有助于理解示例的情况下才使用变量声明。未声明的变量通常是 IDC 函数的输入参数，如 IDA 内置帮助系统中的相应参考页面中命名的那样。

请记住，IDC 在过去几年中已经发生了显著的变化。在其早期版本中，IDC 的主要目的是向脚本程序员公开 SDK 中一些更常用的功能。随着语言功能的增加，已经添加了新的 IDC 函数，其唯一目的是支持高级 IDC 功能，如对象和异常。所有 IDC 函数最终都由 SDK 函数支持，因此，在某种程度上发生了角色反转，新的 IDC 功能要求添加新的 SDK 功能。SDK 的最新版本现在包括一些旨在提供 IDC 对象模型低级实现的函数。在大多数情况下，用户不太可能需要在编译模块内使用这些函数。你可能发现对象操作函数有用的一个例子是当你发现自己正在开发将添加新函数来扩展 IDC 语言的插件时。

| IDC 功能 | SDK 实现 |
| --- | --- |
| `AddAutoStkPnt2` | `add_auto_stkpnt2(get_func(func_ea), ea, delta);` |
| `AddBpt` | `//宏定义用于 AddBptEx(ea, 0, BPT_SOFT);` |
| `AddBptEx` | `add_bpt(ea, size, bpttype);` |
| `AddCodeXref` | `add_cref(From, To, flowtype);` |
| `AddConstEx` | `add_const(enum_id, name, value, bmask);` |
| `AddEntryPoint` | `add_entry(ordinal, ea, name, makecode);` |
| `AddEnum` | `add_enum(idx, name, flag);` |
| `AddHotkey` | `add_idc_hotkey(hotkey, idcfunc);` |
| `AddSeg` |

```
segment_t s;
s.startEA = startea;
s.endEA = endEA;
s.sel = setup_selector(base);
s.bitness = use32;
s.align = align;
s.comb = comb;
add_segm_ex(&s, NULL, NULL, ADDSEG_NOSREG);
```

|

| `AddSourceFile` | `add_sourcefile(ea1, ea2, filename);` |
| --- | --- |
| `AddStrucEx` | `add_struc(index, name, is_union);` |
| `AddStrucMember` |

```
typeinfo_t mt;
//calls an internal function to initialize mt using typeid
add_struc_member(get_struc(id), name, offset, flag, &mt, nbytes);
```

|

| `AltOp` |
| --- |

```
get_forced_operand(ea, n, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `Analysis` | `//macro for SetCharPrm(INF_AUTO, x)` |
| --- | --- |
| `AnalyzeArea` | `analyze_area(sEA, eEA);` |
| `Appcall` |

```
//nargs is the number of arguments following type
//args is idc_value_t[] of args following type
idc_value_t result;
if (type.vtype == VT_LONG && type.num == 0)
   appcall(ea, 0, NULL, NULL, nargs, args, &result);
else
   idc_value_t tval, fields;
   internal_parse_type(&type, &tval, &fields);
   appcall(ea, 0, &tval, &fields, nargs, args, &result);
```

|

| `AppendFchunk` | `append_func_tail(get_func(funcea), ea1, ea2);` |
| --- | --- |
| `ApplySig` | `plan_to_apply_idasgn(name);` |
| `AskAddr` |

```
ea_t addr = defval;
askaddr(&addr, "%s", prompt):
return addr;
```

|

| `AskFile` | `return qstrdup(askfile_c(forsave, mask, "%s", prompt));` |
| --- | --- |
| `AskIdent` | `return qstrdup(askident(defval, "%s", prompt));` |
| `AskLong` |

```
sval_t val = defval;
asklong(&val, "%s", prompt):
return val;
```

|

| `AskSeg` |
| --- |

```
sel_t seg = defval;
askseg(&sel, "%s", prompt):
return val;
```

|

| `AskSelector` | `return ask_selector(sel);` |
| --- | --- |
| `AskStr` | `return qstrdup(askstr(HIST_CMT, defval, "%s", prompt));` |
| `AskYN` | `return askyn_c(defval, "%s", prompt);` |
| `AttachProcess` | `return attach_process(pid, event_id);` |
| `AutoMark` | `//macro, see AutoMark2` |
| `AutoMark2` | `auto_mark_range(start, end, queuetype);` |
| `AutoShow` | `//macro, see SetCharPrm` |
| `AutoUnmark` |

```
//*** undocumented function
autoUnmark(start, end, type);
```

|

| `Batch` | `::batch = batch;` |
| --- | --- |
| `BeginEA` | `//macro, see GetLongPrm` |
| `BeginTypeUpdating` | `return begin_type_updating(utp)` |
| `Byte` | `return get_full_byte(ea);` |
| `CanExceptionContinue` | `return get_debug_event()->can_cont;` |
| `ChangeConfig` | `internal_change_config(line)` |
| `CheckBpt` | `check_bpt(ea)` |
| `Checkpoint` | `//*** undocumented function` |
| `ChooseFunction` | `return choose_func(ea, −1)->startEA;` |
| `CleanupAppcall` | `return cleanup_appcall(0) == 0;` |
| `CmtIndent` | `//macro, see SetCharPrm` |
| `CommentEx` |

```
get_cmt(ea, repeatable, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `Comments` | `//macro, see SetCharPrm` |
| --- | --- |
| `Compile` | `//macro for CompileEx(file, 1);` |
| `CompileEx` |

```
if (isfile)
   CompileEx(input, CPL_DEL_MACROS &#124; CPL_USE_LABELS,
            errbuf, sizeof(errbuf));
else
   CompileLineEx(input, errbuf, sizeof(errbuf));
```

|

| `CreateArray` |
| --- |

```
qsnprintf(buf, sizeof(buf), "$ idc_array %s", name);
netnode n(buf, 0, true);
return (nodeidx_t)n;
```

|

| `DbgByte` |
| --- |

```
if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
   uint8_t b;
   dbg->read_memory(ea, &b, sizeof(b));
   return b;
```

|

| `DbgDword` |
| --- |

```
if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
   uint32_t d;
   dbg->read_memory(ea, &d, sizeof(d));
   return d;
```

|

| `DbgQword` |
| --- |

```
if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
   uint64_t q;
   dbg->read_memory(ea, &q, sizeof(q));
   return q;
```

|

| `DbgRead` |
| --- |

```
if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
   uint8_t *buf = (uint8_t*) qalloc(len);
   dbg->read_memory(ea, buf, len);
   return buf;
```

|

| `DbgWord` |
| --- |

```
if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
   uint16_t w;
   dbg->read_memory(ea, &w, sizeof(w));
   return w;
```

|

| `DbgWrite` |
| --- |

```
if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
   dbg->write_memory(ea, data, length of data);
```

|

| `DecodeInstruction` |
| --- |

```
ua_ana0(ea);
return cmd;
```

|

| `DefineException` | `return define_exception(code, name, desc, flags);` |
| --- | --- |
| `DelArrayElement` | `netnode n(id).supdel(idx, tag);` |
| `DelBpt` | `del_bpt(ea);` |
| `DelCodeXref` | `del_cref(From, To, undef);` |
| `DelConstEx` | `del_const(enum_id, value, serial, bmask);` |
| `DelEnum` | `del_enum(enum_id);` |
| `DelExtLnA` | `netnode n(ea).supdel(n + 1000);` |
| `DelExtLnB` | `netnode n(ea).supdel(n + 2000);` |
| `DelFixup` | `del_fixup(ea);` |
| `DelFunction` | `del_func(ea);` |
| `DelHashElement` |

```
netnode n(id);
n.hashdel(idx);
```

|

| `DelHiddenArea` | `del_hidden_area (ea);` |
| --- | --- |
| `DelHotkey` | `del_idc_hotkey(hotkey);` |
| `DelLineNumber` | `del_source_linnum(ea);` |
| `DelSeg` | `del_segm(ea, flags);` |
| `DelSelector` | `del_selector(sel);` |
| `DelSourceFile` | `del_sourcefile(ea);` |
| `DelStkPnt` | `del_stkpnt(get_func(func_ea), ea);` |
| `DelStruc` | `del_struc(get_struc(id));` |
| `DelStrucMember` | `del_struc_member(get_struc(id), offset);` |
| `DelXML` | `del_xml(path);` |
| `DeleteAll` |

```
while (get_segm_qty ())
   del_segm(getnseg (0), 0);
FlagsDisable(0, inf.ominEA);
FlagsDisable(inf.omaxEA, 0xFFFFFFFF);
```

|

| `DeleteArray` | `netnode n(id).kill();` |
| --- | --- |
| `Demangle` |

```
demangle_name(buf, sizeof(buf), name, disable_mask);
return qstrdup(buf);
```

|

| `DetachProcess` | `detach_process();` |
| --- | --- |
| `Dfirst` | `return get_first_dref_from(From);` |
| `DfirstB` | `return get_first_dref_to(To);` |
| `Dnext` | `return get_next_dref_from(From, current);` |
| `DnextB` | `return get_next_dref_to(To, current);` |
| `Dword` | `return get_full_long(ea);` |
| `EnableBpt` | `enable_bpt(ea, enable);` |
| `EnableTracing` |

```
if (trace_level == 0)
   return enable_step_trace(enable);
else if (trace_level == 1)
   return enable_insn_trace(enable);
else if (trace_level == 2)
   return enable_func_trace(enable);
```

|

| `EndTypeUpdating` | `end_type_updating(utp);` |
| --- | --- |
| `Eval` | `idc_value_t v; calcexpr(-1, expr, &v, errbuf, sizeof(errbuf));` |
| `Exec` | `call_system(command);` |
| `ExecIDC` |

```
char fname[16];
uint32_t fnum = globalCount++; //mutex around globalCount
qsnprintf(fname, sizeof(fname), "___idcexec%d", fnum);
uint32_t len;
len = qsnprintf(NULL, 0, "static %s() {\n%s\n; }", fname, input);
char *func = (char*)qalloc(len);
qsnprintf(func, len, "static %s() {\n%s\n; }", fname, input);
ExecuteLine(func, fname, NULL, 0, NULL, NULL, err, sizeof(err));
globalCount--; //mutex around globalCount
qfree(func);
```

|

| `Exit` | `qexit(code);` |
| --- | --- |
| `ExtLinA` |

```
netnode n(ea).supset(n + 1000, line);
setFlbits(ea, FF_LINE);
```

|

| `ExtLinB` |
| --- |

```
netnode n(ea).supset(n + 2000, line);
setFlbits(ea, FF_LINE);
```

|

| `Fatal` | `error(format, ...);` |
| --- | --- |
| `FindBinary` |

```
ea_t endea = (flag & SEARCH_DOWN) ? inf.maxEA : inf.minEA;
return find_binary(ea, endea, str, getDefaultRadix(), flag);
```

|

| `FindCode` | `return find_code(ea, flag);` |
| --- | --- |
| `FindData` | `return find_data(ea, flag);` |
| `FindExplored` | `return find_defined(ea, flag);` |
| `FindFuncEnd` |

```
func_t f;
find_func_bounds(ea, &f, FIND_FUNC_DEFINE);
return f->endEA;
```

|

| `FindImmediate` | `return find_imm(ea, flag, value);` |
| --- | --- |
| `FindSelector` | `return find_selector(val);` |
| `FindText` | `return find_text(ea, y, x, str, flag);` |
| `FindUnexplored` | `return find_unknown(ea, flag);` |
| `FindVoid` | `return find_void(ea, flag);` |
| `FirstFuncFchunk` | `get_func(funcea)->startEA;` |
| `FirstSeg` | `return getnseg (0)->startEA;` |
| `ForgetException` |

```
excvec_t *ev = retrieve_exceptions();
for (excvec_t::iterator i = ev->begin(); i != ev->end(); i++)
   if ((*i).code == code)
      ev->erase(i);
      return store_exceptions();
return 0;
```

|

| `GenCallGdl` | `gen_simple_call_chart(outfile, "Building graph", title, flags);` |
| --- | --- |
| `GenFuncGdl` |

```
func_t *f = get_func(ea1);
gen_flow_graph(outfile, title, f, ea1, ea2, flags);
```

|

| `GenerateFile` | `gen_file(type, file_handle, ea1, ea2, flags);` |
| --- | --- |
| `GetArrayElement` |

```
netnode n(id);
if (tag == 'A') return n.altval(idx);
else if (tag == 'S')
   n.supstr(idx, buf, sizeof(buf));
   return qstrdup(buf);
```

|

| `GetArrayId` |
| --- |

```
qsnprintf(buf, sizeof(buf), "$ idc_array %s", name);
netnode n(buf);
return (nodeidx_t)n;
```

|

| `GetBmaskCmt` |
| --- |

```
get_bmask_cmt(enum_id, bmask, repeatable, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetBmaskName` |
| --- |

```
get_bmask_name(enum_id, bmask, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetBptAttr` |
| --- |

```
bpt_t bpt;
if (get_bpt(ea, &bpt) == 0) return −1;
if (bpattr == BPTATTR_EA) return bpt.ea;
else if (bpattr == BPTATTR_SIZE) return bpt.size;
else if (bpattr ==BPTATTR_TYPE) return bpt.type;
else if (bpattr == BPTATTR_COUNT) return bpt.pass_count;
else if (bpattr == BPTATTR_FLAGS) return bpt.flags;
else if (bpattr == BPTATTR_COND) return qstrdup(bpt.condition);
```

|

| `GetBptEA` |
| --- |

```
bpt_t bpt;
return getn_bpt(n, &bpt) ? bpt.ea : −1;
```

|

| `GetBptQty` | `return get_bpt_qty();` |
| --- | --- |
| `GetCharPrm` |

```
if (offset <= 191)
   return *(unsigned char*)(offset + (char*)&inf);
```

|

| `GetColor` |
| --- |

```
if (what == CIC_ITEM)
   return get_color(ea);
else if (what == CIC_FUNC)
   return get_func(ea)->color;
else if (what == CIC_SEGM)
   return get_seg(ea)->color;
return 0xFFFFFFFF;
```

|

| `GetConstBmask` | `return get_const_bmask(const_id);` |
| --- | --- |
| `GetConstByName` | `return get_const_by_name(name);` |
| `GetConstCmt` |

```
get_const_cmt(const_id, repeatable, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetConstEnum` | `return get_const_enum(const_id);` |
| --- | --- |
| `GetConstEx` | `return get_const(enum_id, value, serial, bmask);` |
| `GetConstName` |

```
get_const_name(const_id, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetConstValue` | `return get_const_value(const_id);` |
| --- | --- |
| `GetCurrentLine` |

```
tag_remove(get_curline(), buf, sizeof(buf))
return qstrdup(buf);
```

|

| `GetCurrentThreadId` | `return get_current_thread();` |
| --- | --- |
| `GetCustomDataFormat` | `return find_custom_data_format(name);` |
| `GetCustomDataType` | `return find_custom_data_type(name);` |
| `GetDebuggerEvent` | `return wait_for_next_event(wfne, timeout);` |
| `GetDisasm` |

```
generate_disasm_line(ea, buf, sizeof(buf));
tag_remove(buf, buf, 0);
return qstrdup(buf);
```

|

| `GetEntryName` |
| --- |

```
get_entry_name(ordinal, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetEntryOrdinal` | `return get_entry_ordinal(index);` |
| --- | --- |
| `GetEntryPoint` | `return get_entry(ordinal);` |
| `GetEntryPointQty` | `return get_entry_qty();` |
| `GetEnum` | `return get_enum(name);` |
| `GetEnumCmt` |

```
get_enum_cmt(enum_id, repeatable, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetEnumFlag` | `return get_enum_flag(enum_id);` |
| --- | --- |
| `GetEnumIdx` | `return get_enum_idx(enum_id);` |
| `GetEnumName` |

```
get_enum_name(enum_id, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetEnumQty` | `return get_enum_qty();` |
| --- | --- |
| `GetEnumSize` | `return get_enum_size(enum_id);` |
| `GetEnumWidth` |

```
if (enum_id > 0xff000000)
   netnode n(enum_id);
   return (n.altval(0xfffffffb) >> 3) & 7;
else
   return 0;
```

|

| `GetEventBptHardwareEa` | `return get_debug_event()->bpt.hea;` |
| --- | --- |
| `GetEventEa` | `return get_debug_event()->ea;` |
| `GetEventExceptionCode` | `return get_debug_event()->exc.code;` |
| `GetEventExceptionEa` | `return get_debug_event()->exc.ea;` |
| `GetEventExceptionInfo` | `return qstrdup(get_debug_event()->exc.info);` |
| `GetEventExitCode` | `return get_debug_event()->exit_code;` |
| `GetEventId` | `return get_debug_event()->eid;` |
| `GetEventInfo` | `return qstrdup(get_debug_event()->info);` |
| `GetEventModuleBase` | `return get_debug_event()->modinfo.base;` |
| `GetEventModuleName` | `return qstrdup(get_debug_event()->modinfo.name);` |
| `GetEventModuleSize` | `return get_debug_event()->modinfo.size;` |
| `GetEventPid` | `return get_debug_event()->pid;` |
| `GetEventTid` | `return get_debug_event()->tid;` |
| `GetExceptionCode` |

```
excvec_t *ev = retrieve_exceptions();
return idx < ev->size() ? (*ev)[idx].code : 0;
```

|

| `GetExceptionFlags` |
| --- |

```
excvec_t *ev = retrieve_exceptions();
for (excvec_t::iterator i = ev->begin(); i != ev->end(); i++)
   if ((*i).code == code)
      return (*i).flags;
return −1;
```

|

| `GetExceptionName` |
| --- |

```
excvec_t *ev = retrieve_exceptions();
for (excvec_t::iterator i = ev->begin(); i != ev->end(); i++)
   if ((*i).code == code)
      return new qstring((*i).name);
return NULL;
```

|

| `GetExceptionQty` | `return retrieve_exceptions()->size();` |
| --- | --- |
| `GetFchunkAttr` |

```
func_t *f = get_func(ea);
return internal_get_attr(f, attr);
```

|

| `GetFchunkReferer` |
| --- |

```
func_t *f = get_fchunk(ea);
func_parent_iterator_t fpi(f);
return n < f->refqty ? f->referers[n] : BADADDR;
```

|

| `GetFirstBmask` | `return get_first_bmask(enum_id);` |
| --- | --- |
| `GetFirstConst` | `return get_first_const(enum_id, bmask);` |
| `GetFirstHashKey` |

```
netnode n(id).hash1st(buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetFirstIndex` | `return netnode n(id).sup1st(tag);` |
| --- | --- |
| `GetFirstMember` | `return get_struc_first_offset(get_struc(id));` |
| `GetFirstModule` |

```
module_info_t modinfo;
get_first_module(&modinfo);
return modinfo.base;
```

|

| `GetFirstStrucIdx` | `return get_first_struc_idx();` |
| --- | --- |
| `GetFixupTgtDispl` |

```
fixup_data_t fd;
get_fixup(ea, &fd);
return fd.displacement;
```

|

| `GetFixupTgtOff` |
| --- |

```
fixup_data_t fd;
get_fixup(ea, &fd);
return fd.off
```

|

| `GetFixupTgtSel` |
| --- |

```
fixup_data_t fd;
get_fixup(ea, &fd);
return fd.sel;
```

|

| `GetFixupTgtType` |
| --- |

```
fixup_data_t fd;
get_fixup(ea, &fd);
return fd.type;
```

|

| `GetFlags` | `getFlags(ea);` |
| --- | --- |
| `GetFpNum` |

```
//*** undocumented function
char buf[16];
union {float f; double d; long double ld} val;
get_many_bytes(ea, buf, len > 16 ? 16 : len);
ph.realcvt(buf, &val, (len >> 1) - 1);
return val;
```

|

| `GetFrame` | `//macro, see GetFunctionAttr` |
| --- | --- |
| `GetFrameArgsSize` | `//macro, see GetFunctionAttr` |
| `GetFrameLvarSize` | `//macro, see GetFunctionAttr` |
| `GetFrameRegsSize` | `//macro, see GetFunctionAttr` |
| `GetFrameSize` | `return get_frame_size(get_func(ea));` |
| `GetFuncOffset` |

```
int flags =  GNCN_REQFUNC &#124; GNCN_NOCOLOR;
get_nice_colored_name(ea, buf, sizeof(buf),flags);
return qstrdup(buf);
```

|

| `GetFunctionAttr` |
| --- |

```
func_t *f = get_func(ea);
return internal_get_attr(f, attr);
```

|

| `GetFunctionCmt` | `return get_func_cmt(get_func(ea), repeatable);` |
| --- | --- |
| `GetFunctionFlags` | `//macro, see GetFunctionAttr` |
| `GetFunctionName` |

```
get_func_name(ea, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetHashLong` | `netnode n(id).hashval_long(idx);` |
| --- | --- |
| `GetHashString` |

```
netnode n(id).hashval(idx, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetIdaDirectory` |
| --- |

```
qstrncpy(buf, idadir(NULL), sizeof(buf));
return qstrdup(buf);
```

|

| `GetIdbPath` |
| --- |

```
qstrncpy(buf, database_idb, sizeof(buf));
return qstrdup(buf);
```

|

| `GetInputFile` |
| --- |

```
get_root_filename(buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetInputFilePath` |
| --- |

```
RootNode.valstr(buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetInputMD5` |
| --- |

```
uint8_t md5bin[16];
char out[1024];
char *outp = out;
int len = sizeof(out);
out[0] = 0;
RootNode.supval(RIDX_MD5, md5bin, sizeof(md5bin));
for (int j = 0; j < sizeof(md5bin); j++) {
   int nbytes = qsnprintf(out, len, "%02X", md5bin[j]);
   outp += nbytes;
   len -= nbytes;
}
return qstrdup(out);
```

|

| `GetLastBmask` | `return get_last_bmask(enum_id);` |
| --- | --- |
| `GetLastConst` | `return get_last_const(enum_id, bmask);` |
| `GetLastHashKey` |

```
netnode n(id).hashlast(buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetLastIndex` | `return netnode n(id).suplast(tag);` |
| --- | --- |
| `GetLastMember` | `return get_struc_last_offset(get_struc(id));` |
| `GetLastStrucIdx` | `return get_last_struc_idx();` |
| `GetLineNumber` | `return get_source_linnum(ea);` |
| `GetLocalType` |

```
const type_t *type;
const p_list *fields;
get_numbered_type(idati, ordinal, &type, &fields,
                  NULL, NULL, NULL);
char *name = get_numbered_type_name(idati, ordinal);
qstring res;
print_type_to_qstring(&res, 0, 2, 40, flags, idati, type,
                      name, NULL, fields, NULL);
return qstrdup(res.c_str());
```

|

| `GetLocalTypeName` | `return qstrdup(get_numbered_type_name(idati, ordinal));` |
| --- | --- |
| `GetLongPrm` |

```
if (offset <= 188)
   return *(int*)(offset + (char*)&inf);
```

|

| `GetManualInsn` |
| --- |

```
get_manual_insn(ea, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetManyBytes` |
| --- |

```
uint8_t *out = (uint8_t*)qalloc(size + 1);
if (use_dbg)
   if (dbg && (dbg->may_disturb() &#124;&#124; get_process_state() < 0))
      dbg->read_memory(ea, out, size);
   else
      qfree(out);
      out = NULL;
else
   get_many_bytes(ea, out, size);
return out;
```

|

| `GetMarkComment` |
| --- |

```
curloc loc.markdesc(slot, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetMarkedPos` | `return curloc loc.markedpos(&slot);` |
| --- | --- |
| `GetMaxLocalType` | `return get_ordinal_qty(idati);` |
| `GetMemberComment` |

```
tid_t m = get_member(get_struc(id), offset)->id;
netnode n(m).supstr(repeatable ? 1 : 0, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetMemberFlag` | `return get_member(get_struc(id), offset)->flag;` |
| --- | --- |
| `GetMemberName` |

```
tid_t m = get_member(get_struc(id), offset)->id;
get_member_name(m, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetMemberOffset` | `return get_member_by_name(get_struc(id), member_name)->soff;` |
| --- | --- |
| `GetMemberQty` | `get_struc(id)->memqty;` |
| `GetMemberSize` |

```
member_t *m = get_member(get_struc(id), offset);
return get_member_size(m);
```

|

| `GetMemberStrId` |
| --- |

```
tid_t m = get_member(get_struc(id), offset)->id;
return netnode n(m).altval(3) - 1;
```

|

| `GetMinSpd` |
| --- |

```
func_t *f = get_func(ea);
return f ? get_min_spd_ea(f) : BADADDR;
```

|

| `GetMnem` |
| --- |

```
ua_mnem(ea, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetModuleName` |
| --- |

```
module_info_t modinfo;
if (base == 0)
   get_first_module(&modinfo);
else
   modinfo.base = base - 1;
   get_next_module(&modinfo);
return qstrdup(modinfo.name);
```

|

| `GetModuleSize` |
| --- |

```
module_info_t modinfo;
if (base == 0)
   get_first_module(&modinfo);
else
   modinfo.base = base - 1;
   get_next_module(&modinfo);
return modinfo.size;
```

|

| `GetNextBmask` | `return get_next_bmask(eum_id, value);` |
| --- | --- |
| `GetNextConst` | `return get_next_const(enum_id, value, bmask);` |
| `GetNextFixupEA` | `return get_next_fixup_ea(ea);` |
| `GetNextHashKey` |

```
netnode n(id).hashnxt(idx, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetNextIndex` | `return netnode n(id).supnxt(idx, tag);` |
| --- | --- |
| `GetNextModule` |

```
module_info_t modinfo;
modinfo.base = base;
get_next_module(&modinfo);
return modinfo.base;
```

|

| `GetNextStrucIdx` | `return get_next_struc_idx();` |
| --- | --- |
| `GetOpType` |

```
*buf = 0;
if (isCode(get_flags_novalue(ea)))
   ua_ana0(ea);
   return cmd.Operands[n].type;
```

|

| `GetOperandValue` |
| --- |

```
Use ua_ana0 to fill command struct then return
appropriate value based on cmd.Operands[n].type
```

|

| `GetOpnd` |
| --- |

```
*buf = 0;
if (isCode(get_flags_novalue(ea)))
   ua_outop2(ea, buf, sizeof(buf), n);
tag_remove(buf, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetOriginalByte` | `return get_original_byte(ea);` |
| --- | --- |
| `GetPrevBmask` | `return get_prev_bmask(enum_id, value);` |
| `GetPrevConst` | `return get_prev_const(enum_id, value, bmask);` |
| `GetPrevFixupEA` | `return get_prev_fixup_ea(ea);` |
| `GetPrevHashKey` |

```
netnode n(id).hashprev(idx, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetPrevIndex` | `return netnode n(id).supprev(idx, tag);` |
| --- | --- |
| `GetPrevStrucIdx` | `return get_prev_struc_idx(index);` |
| `GetProcessName` |

```
process_info_t p;
pid_t pid = get_process_info(idx, &p);
return qstrdup(p.name);
```

|

| `GetProcessPid` | `return get_process_info(idx, NULL);` |
| --- | --- |
| `GetProcessQty` | `return get_process_qty();` |
| `GetProcessState` | `return get_process_state();` |
| `GetReg` | `return getSR(ea, str2reg(reg));` |
| `GetRegValue` |

```
regval_t r;
get_reg_val(name, &r);
if (is_reg_integer(name))
   return (int)r.ival;
else
   //memcpy(result, r.fval, 12);
```

|

| `GetSegmentAttr` |
| --- |

```
segment_t *s = get_seg(segea);
return internal_get_attr(s, attr);
```

|

| `GetShortPrm` |
| --- |

```
if (offset <= 190)
   return *(unsigned short*)(offset + (char*)&inf);
```

|

| `GetSourceFile` | `return qstrdup(get_sourcefile(ea));` |
| --- | --- |
| `GetSpDiff` | `return get_sp_delta(get_func(ea), ea);` |
| `GetSpd` | `return get_spd(get_func(ea), ea);` |
| `GetString` |

```
if (len == −1)
   len = get_max_ascii_length(ea, type, true);
get_ascii_contents(ea, len, type, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetStringType` | `return netnode n(ea).altval(16) - 1;` |
| --- | --- |
| `GetStrucComment` |

```
get_struc_cmt(id, repeatable, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetStrucId` | `return get_struc_by_idx(index);` |
| --- | --- |
| `GetStrucIdByName` | `return get_struc_id(name);` |
| `GetStrucIdx` | `return get_struc_idx(id);` |
| `GetStrucName` |

```
get_struc_name(id, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetStrucNextOff` | `return get_struc_next_offset(get_struc(id), offset);` |
| --- | --- |
| `GetStrucPrevOff` | `return get_struc_prev_offset(get_struc(id), offset);` |
| `GetStrucQty` | `return get_struc_qty();` |
| `GetStrucSize` | `return get_struc_size(id);` |
| `GetTestId` | `//*** undocumented, returns internal testId` |
| `GetThreadId` | `return getn_thread(idx);` |
| `GetThreadQty` | `return get_thread_qty();` |
| `GetTinfo` | `//no comparable return type in SDK, generally uses get_tinfo` |
| `GetTrueName` | `//macro, see GetTrueNameEx` |
| `GetTrueNameEx` | `return qstrdup(get_true_name(from, ea, buf, sizeof(buf)));` |
| `GetType` |

```
get_ti(ea, tbuf, sizeof(tbuf), plist, sizeof(plist));
print_type_to_one_line(buf, sizeof(buf), idati,
                       tbuf, NULL, NULL, plist, NULL);
return qstrdup(buf);
```

|

| `GetnEnum` | `return getn_enum(idx);` |
| --- | --- |
| `GetVxdFuncName` |

```
//*** undocumented function
get_vxd_func_name(vxdnum, funcnum, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `GetXML` |
| --- |

```
valut_t res;
get_xml(path, &res);
return res;
```

|

| `GuessType` |
| --- |

```
guess_type(ea, tbuf, sizeof(tbuf), plist, sizeof(plist));
print_type_to_one_line(buf, sizeof(buf), idati, tbuf,
                       NULL, NULL, plist, NULL);
return qstrdup(buf);
```

|

| `HideArea` | `add_hidden_area(start, end, description, header, footer, color);` |
| --- | --- |
| `HighVoids` | `//macro, see SetLongPrm` |
| `IdbByte` | `return get_db_byte(ea);` |
| `Indent` | `//macro, see SetCharPrm` |
| `IsBitfield` | `return is_bf(enum_id);` |
| `IsEventHandled` | `return get_debug_event()->handled;` |
| `IsFloat` | `//IDC variable type query, n/a for SDK` |
| `IsLong` | `//IDC variable type query, n/a for SDK` |
| `IsObject` | `//IDC variable type query, n/a for SDK` |
| `IsString` | `//IDC variable type query, n/a for SDK` |
| `IsUnion` | `return get_struc(id)->is_union();` |
| `ItemEnd` | `return get_item_end(ea);` |
| `ItemHead` | `return get_item_head(ea);` |
| `ItemSize` | `return get_item_end(ea) - ea;` |
| `Jump` | `jumpto(ea);` |
| `LineA` |

```
netnode n(ea).supstr(1000 + num, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `LineB` |
| --- |

```
netnode n(ea).supstr(2000 + num, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `LoadDebugger` | `load_debugger(dbgname, use_remote);` |
| --- | --- |
| `LoadTil` | `return add_til2(name, 0);` |
| `LocByName` | `return get_name_ea(-1, name);` |
| `LocByNameEx` | `return get_name_ea(from, name);` |
| `LowVoids` | `//macro, see SetLongPrm` |
| `MK_FP` | `return ((seg<<4) + off);` |
| `MakeAlign` | `doAlign(ea, count, align);` |
| `MakeArray` |

```
typeinfo_t ti;
flags_t f = get_flags_novalue(ea);
get_typeinfo(ea, 0, f, &ti);
asize_t sz = get_data_elsize(ea, f, &ti);
do_data_ex (ea, f, sz * nitems, ti.tid);
```

|

| `MakeByte` | `//macro, see MakeData` |
| --- | --- |
| `MakeCode` | `ua_code(ea);` |
| `MakeComm` | `set_cmt(ea, cmt, false);` |
| `MakeData` | `do_data_ex(ea, flags, size, tid);` |
| `MakeDouble` | `//macro, see MakeData` |
| `MakeDword` | `//macro, see MakeData` |
| `MakeFloat` | `//macro, see MakeData` |
| `MakeFrame` |

```
func_t *f = get_func(ea);
set_frame_size(f, lvsize, frregs, argsize);
return f->frame;
```

|

| `MakeFunction` | `add_func(start, end);` |
| --- | --- |
| `MakeLocal` |

```
func_t *f = get_func(ea);
if (*location != '[')
   add_regvar(f, start, end, location, name, NULL);
else
   struc_t *fr = get_frame(f);
   int start = f->frsize + offset;
   if (get_member(fr, start))
      set_member_name(fr, start, name);
   else
      add_struc_member(fr, name, start,  0x400, 0, 1);
```

|

| `MakeNameEx` | `set_name(ea, name, flags);` |
| --- | --- |
| `MakeOword` | `//macro, see MakeData` |
| `MakePackReal` | `//macro, see MakeData` |
| `MakeQword` | `//macro, see MakeData` |
| `MakeRptCmt` | `set_cmt(ea, cmt, true);` |
| `MakeStr` |

```
int len = endea == −1 ? 0 : endea - ea;
make_ascii_string(ea, len, current_string_type);
```

|

| `MakeStructEx` |
| --- |

```
netnode n(strname);
nodeidx_t idx = (nodeidx_t)n;
if (size != −1)
   do_data_ex(ea, FF_STRU, size, idx);
else
   size_t sz = get_struc_size(get_struc(idx));
   do_data_ex(ea, FF_STRU, sz, idx);
```

|

| `MakeTbyte` | `//macro, see MakeData` |
| --- | --- |
| `MakeUnkn` | `do_unknown(ea, flags);` |
| `MakeUnknown` | `do_unknown_range(ea, size, flags);` |
| `MakeVar` | `doVar(ea);` |
| `MakeWord` | `//macro, see MakeData` |
| `MarkPosition` |

```
curloc loc;
loc.ea = ea; loc.lnnum = lnnum; loc.x = x; loc.y = y;
loc.mark(slot, NULL, comment);
```

|

| `MaxEA` | `//macro, see GetLongPrm` |
| --- | --- |
| `Message` | `msg(format, ...);` |
| `MinEA` | `//macro, see GetLongPrm` |
| `MoveSegm` | `return move_segm(get_seg(ea), to, flags);` |
| `Name` | `return qstrdup(get_name(-1, ea, buf, sizeof(buf)));` |
| `NameEx` | `return qstrdup(get_name(from, ea, buf, sizeof(buf)));` |
| `NextAddr` | `return nextaddr(ea);` |
| `NextFchunk` | `return funcs->getn_area(funcs->get_next_area(ea))->startEA;` |
| `NextFuncFchunk` | `func_tail_iterator_t fti(get_func(funcea), tailea);``return fti.next() ? fti.chunk().startEA : −1;` |
| `NextFunction` | `return get_next_func(ea)->startEA;` |
| `NextHead` | `return next_head(ea, maxea);` |
| `NextNotTail` | `return next_not_tail(ea);` |
| `NextSeg` |

```
int n = segs.get_next_area(ea);
return getnseg (n)->startEA;
```

|

| `OpAlt` | `set_forced_operand(ea, n, str);` |
| --- | --- |
| `OpBinary` | `op_bin(ea, n);` |
| `OpChr` | `op_chr(ea, n);` |
| `OpDecimal` | `op_dec(ea, n);` |
| `OpEnumEx` | `op_enum(ea, n, enumid, serial);` |
| `OpFloat` | `op_flt(ea, n);` |
| `OpHex` | `op_hex(ea, n);` |
| `OpHigh` | `return op_offset(ea, n, REF_HIGH16, target);` |
| `OpNot` | `toggle_bnot(ea, n);` |
| `OpNumber` | `op_num(ea, n);` |
| `OpOctal` | `op_oct(ea, n);` |
| `OpOff` |

```
if (base != 0xFFFFFFFF) set_offset(ea, n, base);
else noType(ea, n);
```

|

| `OpOffEx` | `op_offset(ea, n, reftype, target, base, tdelta);` |
| --- | --- |
| `OpSeg` | `op_seg(ea, n);` |
| `OpSign` | `toggle_sign(ea, n);` |
| `OpStkvar` | `op_stkvar(ea, n);` |
| `OpStroffEx` | `op_stroff(ea, n, &strid, 1, delta);` |
| `ParseType` |

```
qstring in(input);
if (in.last() != ';') in += ';';
flags &#124;= PT_TYP;
if (flags & PT_NDC) flags &#124;= PT_SIL;
else flags &= ~PT_SIL;
flags &= ~PT_NDC;
qstring name, type, fields;
parse_decl(idati, in.c_str(), &name, &type, &fields, flags);
internal_build_idc_typeinfo(&result, &type, &fields);
```

|

| `ParseTypes` |
| --- |

```
int hti_flags = (flags & 0x70) << 8;
if (flags & 1) hti_flags &#124;= HTI_FIL;
parse_types2(input, (flags & 2) ? NULL : printer_func,
             hti_flags);
```

|

| `PatchByte` | `patch_byte(ea, value);` |
| --- | --- |
| `PatchDbgByte` |

```
if (qthread_same(idc_debthread))
   dbg->write_memory(ea, &value, 1);
else
   put_dbg_byte(ea, value);
```

|

| `PatchDword` | `patch_long(ea, value);` |
| --- | --- |
| `PatchWord` | `patch_word(ea, value);` |
| `PauseProcess` | `suspend_process();` |
| `PopXML` | `pop_xml();` |
| `PrevAddr` | `return prevaddr(ea);` |
| `PrevFchunk` | `return get_prev_fchunk(ea)->startEA;` |
| `PrevFunction` | `return get_prev_func(ea)->startEA;` |
| `PrevHead` | `return prev_head(ea, minea);` |
| `PrevNotTail` | `return prev_not_tail(ea);` |
| `ProcessUiAction` | `return process_ui_action(name, flags);` |
| `PushXML` | `push_xml(path);` |
| `Qword` | `return get_qword(ea);` |
| `RebaseProgram` | `return rebase_program(delta, flags);` |
| `RecalcSpd` | `return recalc_spd(cur_ea);` |
| `Refresh` | `refresh_idaview_anyway();` |
| `RefreshDebuggerMemory` |

```
invalidate_dbgmem_config();
invalidate_dbgmem_contents(BADADDR, −1);
if (dbg && dbg->stopped_at_debug_event)
   dbg->stopped_at_debug_event(true);
```

|

| `RefreshLists` | `callui(ui_list);` |
| --- | --- |
| `RemoveFchunk` | `remove_func_tail(get_func(funcea), tailea);` |
| `RenameArray` |

```
qsnprintf(buf, sizeof(buf), "$ idc_array %s", name);
netnode n(id).rename(newname);
```

|

| `RenameEntryPoint` | `rename_entry(ordinal, name);` |
| --- | --- |
| `RenameSeg` | `set_segm_name(get_seg(ea), "%s", name);` |
| `ResumeThread` | `return resume_thread(tid);` |
| `Rfirst` | `return get_first_cref_from(From);` |
| `Rfirst0` | `return get_first_fcref_from(From);` |
| `RfirstB` | `return get_first_cref_to(To);` |
| `RfirstB0` | `return get_first_fcref_to(To);` |
| `Rnext` | `return get_next_cref_from(From, current);` |
| `Rnext0` | `return get_next_fcref_from(From, current);` |
| `RnextB` | `return get_next_cref_to(To, current);` |
| `RnextB0` | `return get_next_fcref_to(To, current);` |
| `RunPlugin` | `run_plugin(load_plugin(name), arg);` |
| `RunTo` | `run_to(ea);` |
| `SaveBase` |

```
char *fname = idbname ? idbname : database_idb;
uint32_t tflags = database_flags;
database_flags = (flags & 4) &#124; (tflags & 0xfffffffb);
bool res = save_database(fname, 0);
database_flags = tflags;
return res;
```

|

| `ScreenEA` | `return get_screen_ea();` |
| --- | --- |
| `SegAddrng` | `//已弃用，请参阅 SetSegAddressing` |
| `SegAlign` | `//宏定义，请参阅 SetSegmentAttr` |
| `SegBounds` | `//已弃用，请参阅 SetSegBounds` |
| `SegByBase` | `return get_segm_by_sel(base)->startEA;` |
| `SegByName` |

```
sel_t seg;
atos(segname, *seg);
return seg;
```

|

| `SegClass` | `//已弃用，请参阅 SetSegClass` |
| --- | --- |
| `SegComb` | `//宏定义，请参阅 SetSegmentAttr` |
| `SegCreate` | `//已弃用，请参阅 AddSeg` |
| `SegDefReg` | `//已弃用，请参阅 SetSegDefReg` |
| `SegDelete` | `//已弃用，请参阅 DelSeg` |
| `SegEnd` | `//宏定义，请参阅 GetSegmentAttr` |
| `SegName` |

```
segment_t *s = (segment_t*) get_seg(ea);
get_true_segm_name(s, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `SegRename` | `//已弃用，请参阅 RenameSeg` |
| --- | --- |
| `SegStart` | `//宏定义，请参阅 GetSegmentAttr` |
| `SelEnd` |

```
ea_t ea1, ea2;
read_selection(&ea1, &ea2);
return ea2;
```

|

| `SelStart` |
| --- |

```
ea_t ea1, ea2;
read_selection(&ea1, &ea2);
return ea1;
```

|

| `SelectThread` | `select_thread(tid);` |
| --- | --- |
| `SetArrayFormat` |

```
segment_t *s = get_seg(ea);
if (s)
   uint32_t format[3];
   netnode array(ea);
   format[0] = flags;
   format[1] = litems;
   format[2] = align;
   array.supset(5, format, sizeof(format));
```

|

| `SetArrayLong` | `netnode n(id).altset(idx, value);` |
| --- | --- |
| `SetArrayString` | `netnode n(id).supset(idx, str);` |
| `SetBmaskCmt` | `set_bmask_cmt(enum_id, bmask, cmt, repeatable);` |
| `SetBmaskName` | `set_bmask_name(enum_id, bmask, name);` |
| `SetBptAttr` |

```
bpt_t bpt;
if (get_bpt(ea, &bpt) == 0) return;
if (bpattr == BPTATTR_SIZE) bpt.size = value;
else if (bpattr == BPTATTR_TYPE) bpt.type = value;
else if (bpattr == BPTATTR_COUNT) bpt.pass_count = value;
else if (bpattr == BPTATTR_FLAGS) bpt.flags = value;
update_bpt(&bpt);
```

|

| `SetBptCnd` | `//宏定义，用于 SetBptCndEx(ea, cnd, 0);` |
| --- | --- |
| `SetBptCndEx` |

```
bpt_t bpt;
if (get_bpt(ea, &bpt) == 0) return;
bpt. cndbody = cnd;
if (is_lowcnd)
   bpt.flags &#124;= BPT_LOWCND;
else
   bpt.flags &= ~ BPT_LOWCND;
update_bpt(&bpt);
```

|

| `SetCharPrm` |
| --- |

```
if (offset >= 13 && offset <= 191)
   *(offset + (char*)&inf) = value;
```

|

| `SetColor` |
| --- |

```
if (what == CIC_ITEM)
   set_item_color(ea, color);
else if (what == CIC_FUNC)
   func_t *f = get_func(ea);
   f->color = color;
   update_func(f);
else if (what == CIC_SEGM)
   segment_t *s = get_seg(ea);
   s->color = color;
   s->update();
```

|

| `SetConstCmt` | `set_const_cmt(const_id, cmt, repeatable);` |
| --- | --- |
| `SetConstName` | `set_const_name(const_id, name);` |
| `SetDebuggerOptions` | `return set_debugger_options(options);` |
| `SetEnumBf` | `set_enum_bf(enum_id, flag ? 1 : 0);` |
| `SetEnumCmt` | `set_enum_cmt(enum_id, cmt, repeatable);` |
| `SetEnumFlag` | `set_enum_flag(enum_id, flag);` |
| `SetEnumIdx` | `set_enum_idx(enum_id, idx);` |
| `SetEnumName` | `set_enum_name(enum_id, name);` |
| `SetEnumWidth` | `return set_enum_width(enum_id, width);` |
| `SetExceptionFlags` |

```
excvec_t *ev = retrieve_exceptions();
for (excvec_t::iterator i = ev->begin(); i != ev->end(); i++)
   if ((*i).code == code)
      if ((*i).flags == flags)
         return true;
      else
         (*i).flags = flags;
         return store_exceptions();
return 0;
```

|

| `SetFchunkAttr` |
| --- |

```
func_t *f = get_func(ea);
internal_set_attr(f, attr, value);
update_func(f);
```

|

| `SetFchunkOwner` | `set_tail_owner(get_func(tailea), funcea);` |
| --- | --- |
| `SetFixup` |

```
fixup_data_t f = {type, targetsel, targetoff, displ};
set_fixup(ea, &f);
```

|

| `SetFlags` | `setFlags(ea, flags);` |
| --- | --- |
| `SetFunctionAttr` |

```
func_t *f = get_func(ea);
internal_set_attr(f, attr, value);
```

|

| `SetFunctionCmt` | `set_func_cmt (get_func(ea), cmt, repeatable);` |
| --- | --- |
| `SetFunctionEnd` | `func_setend(ea, end);` |
| `SetFunctionFlags` | `//macro, see SetFunctionAttr` |
| `SetHashLong` | `netnode n(id).hashset(idx, value);` |
| `SetHashString` | `netnode n(id).hashset(idx, value);` |
| `SetHiddenArea` |

```
hidden_area_t *ha = get_hidden_area (ea);
ha->visible = visible;
update_hidden_area(ha);
```

|

| `SetInputFilePath` |
| --- |

```
if (strlen(path) == 0) RootNode.set("");
else RootNode.set(path);
```

|

| `SetLineNumber` | `set_source_linnum(ea, lnnum);` |
| --- | --- |
| `SetLocalType` |

```
if (input == NULL &#124;&#124; *input == 0)
   del_numbered_type(idati, ordinal);
else
   qstring name;
   qtype type, fields;
   parse_decl(idati, input, &name, &type, &fields, flags);
   if (ordinal == 0)
      if (!name.empty())
         get_named_type(idati, name.c_str(),
                        NTF_TYPE &#124; NTF_NOBASE, NULL, NULL,
                        NULL, NULL, NULL, &ordinal);
         if (!ordinal)
            ordinal = alloc_type_ordinal(idati);
   set_numbered_type(idati, value, 0, name.c_str(),
                     type.c_str(), fields.c_str(),
                     NULL, NULL, NULL);
```

|

| `SetLongPrm` |
| --- |

```
if (offset >= 13 && offset <= 188)
   *(int*)(offset + (char*)&inf) = value;
```

|

| `SetManualInsn` | `set_manual_insn(ea, insn);` |
| --- | --- |
| `SetMemberComment` |

```
member_t *m = get_member(get_struc(ea), member_offset);
set_member_cmt(m, comment, repeatable);
```

|

| `SetMemberName` | `set_member_name(get_struc(ea), member_offset, name);` |
| --- | --- |
| `SetMemberType` |

```
typeinfo_t mt;
//calls an internal function to initialize mt using typeid
int size =  get_data_elsize(-1, flag, &mt) *  nitems;
set_member_type(get_struc(id), member_offset, flag, &mt,size);
```

|

| `SetProcessorType` | `set_processor_type(processor, level);` |
| --- | --- |
| `SetReg` | `//macro for SetRegEx(ea, reg, value, SR_user);` |
| `SetRegEx` | `splitSRarea1(ea, str2reg(reg), value, tag, false);` |
| `SetRegValue` |

```
regval_t r;
if (is_reg_integer(name))
   r.ival = (unsigned int)VarLong(value);
else
   memcpy(r.fval, VarFloat(value), 12);
set_reg_val(name, &r);
```

|

| `SetRemoteDebugger` | `set_remote_debugger(hostname, password, portnum);` |
| --- | --- |
| `SetSegAddressing` | `set_segm_addressing(get_seg(ea), use32);` |
| `SetSegBounds` |

```
if (get_seg(ea))
   set_segm_end(ea, endea, flags);
   set_segm_end(ea, startea, flags);
```

|

| `SetSegClass` | `set_segm_class(get_seg(ea), class);` |
| --- | --- |
| `SetSegDefReg` | `SetDefaultRegisterValue(get_seg(ea), str2reg(reg), value);` |
| `SetSegmentAttr` |

```
segment_t *s = get_seg(segea);
internal_set_attr(s, attr, value);
s->update();
```

|

| `SetSegmentType` | `//macro, see SetSegmentAttr` |
| --- | --- |
| `SetSelector` | `set_selector(sel, value);` |
| `SetShortPrm` |

```
if (offset >= 13 && offset <= 190)
   *(short*)(offset + (char*)&inf) = value;
```

|

| `SetSpDiff` | `add_user_stkpnt(ea, delta);` |
| --- | --- |
| `SetStatus` | `setStat(status);` |
| `SetStrucComment` | `set_struc_cmt(id, cmt, repeatable);` |
| `SetStrucIdx` | `set_struc_idx(get_struc(id), index);` |
| `SetStrucName` | `set_struc_name(id, name);` |
| `SetTargetAssembler` | `set_target_assembler(asmidx);` |
| `SetType` |

```
apply_cdecl(ea, type)
if (get_aflags(ea) & AFL_TILCMT)
   set_ti(ea, "", NULL);
```

|

| `SetXML` | `set_xml(path, name, value);` |
| --- | --- |
| `Sleep` | `qsleep(milliseconds);` |
| `StartDebugger` | `start_process(path, args, sdir);` |
| `StepInto` | `step_into();` |
| `StepOver` | `step_over();` |
| `StepUntilRet` | `step_until_ret();` |
| `StopDebugger` | `exit_process();` |
| `StringStp` | `//macro, see SetCharPrm` |
| `Tabs` | `//macro, see SetCharPrm` |
| `TakeMemorySnapshot` | `take_memory_snapshot(only_loader_segs);` |
| `TailDepth` | `//macro, see SetLongPrm` |
| `Til2Idb` | `return til2idb(idx, type_name);` |
| `Voids` | `//macro, see SetCharPrm` |
| `Wait` | `autoWait();` |
| `Warning` | `warning(format, ...);` |
| `Word` | `return get_full_word(ea);` |
| `XrefShow` | `//macro, see SetCharPrm` |
| `XrefType` | `Returns value of an internal global variable` |
| `____` |

```
//*** undocumented function (four underscores)
//returns database creation timestamp
return RootNode.altval(RIDX_ALT_CTIME);
```

|

| `_call` |
| --- |

```
//*** undocumented function
//uint32_t _call(uint32_t (*f)())
//f is a pointer in IDA's (NOT the database's) address space
return (*f)();
```

|

| `_lpoke` |
| --- |

```
//*** undocumented function
//uint32_t _lpoke(uint32_t *addr, uint32_t val)
//addr is an address in IDA's (NOT the database's) address
//space. This modifies IDA’s address space NOT the database’s
uint32_t old = *addr;
*addr = val;
return old;
```

|

| `_peek` |
| --- |

```
//*** undocumented function
//uint8_t *_peek(uint8_t *addr)
//addr is in IDA's address space
return *addr;
```

|

| `_poke` |
| --- |

```
//*** undocumented function
//uint8_t _lpoke(uint8_t *addr, uint8_t val)
//addr is an address in IDA's (NOT the database's) address
//space. This modifies IDA's address space NOT the database's
uint8_t old = *addr;
*addr = val;
return old;
```

|

| `_time` |
| --- |

```
//*** undocumented function
return _time64(NULL);
```

|

| `add_dref` | `add_dref(From, To, drefType);` |
| --- | --- |
| `atoa` |

```
ea2str(ea, buf, sizeof(buf));
return qstrdup(buf);
```

|

| `atol` | `return atol(str);` |
| --- | --- |
| `byteValue` | `//macro` |
| `del_dref` | `del_dref(From, To);` |
| `delattr` | `VarDelAttr(self, attr);` |
| `fclose` | `qfclose(handle);` |
| `fgetc` | `return qfgetc(handle);` |
| `filelength` | `return efilelength(handle);` |
| `fopen` | `return qfopen(file, mode);` |
| `form` | `//deprecated, see sprintf` |
| `fprintf` | `qfprintf(handle, format, ...);` |
| `fputc` | `qfputc(byte, handle);` |
| `fseek` | `qfseek(handle, offset, origin);` |
| `ftell` | `return qftell(handle);` |
| `get_field_ea` | `Too complex to summarize` |
| `get_nsec_stamp` | `return get_nsec_stamp();` |
| `getattr` |

```
idc_value_t res;
VarGetAttr(self, attr, &res);
return res;
```

|

| `hasattr` | `return VarGetAttr(self, attr, NULL) == 0;` |
| --- | --- |
| `hasName` | `//macro` |
| `hasValue` | `//macro` |
| `isBin0` | `//macro` |
| `isBin1` | `//macro` |
| `isChar0` | `//macro` |
| `isChar1` | `//macro` |
| `isCode` | `//macro` |
| `isData` | `//macro` |
| `isDec0` | `//macro` |
| `isDec1` | `//macro` |
| `isDefArg0` | `//macro` |
| `isDefArg1` | `//macro` |
| `isEnum0` | `//macro` |
| `isEnum1` | `//macro` |
| `isExtra` | `//macro` |
| `isFlow` | `//macro` |
| `isFop0` | `//macro` |
| `isFop1` | `//macro` |
| `isHead` | `//macro` |
| `isHex0` | `//macro` |
| `isHex1` | `//macro` |
| `isLoaded` | `//macro` |
| `isOct0` | `//macro` |
| `isOct1` | `//macro` |
| `isOff0` | `//macro` |
| `isOff1` | `//macro` |
| `isRef` | `//macro` |
| `isSeg0` | `//macro` |
| `isSeg1` | `//macro` |
| `isStkvar0` | `//macro` |
| `isStkvar1` | `//macro` |
| `isStroff0` | `//macro` |
| `isStroff1` | `//macro` |
| `isTail` | `//macro` |
| `isUnknown` | `//macro` |
| `isVar` | `//macro` |
| `lastattr` | `return qstrdup(VarLastAttr(self));` |
| `loadfile` |

```
linput_t *li = make_linput(handle);
file2base(li, pos, ea, ea + size, false);
unmake_linput(li);
```

|

| `ltoa` | `Calls internal conversion routine` |
| --- | --- |
| `mkdir` | `return qmkdir(dirname, mode);` |
| `nextattr` | `return qstrdup(VarNextAttr(self, attr));` |
| `ord` | `return str[0];` |
| `prevattr` | `return qstrdup(VarPrevAttr(self, attr));` |
| `print` |

```
qstring qs;
VarPrint(&qs, arg);
msg("%s\n", qs.c_str());
```

|

| `readlong` |
| --- |

```
unsigned int res;
freadbytes(handle, &res, 4, mostfirst);
return res;
```

|

| `readshort` |
| --- |

```
unsigned short res;
freadbytes(handle, &res, 2, mostfirst);
return res;
```

|

| `readstr` |
| --- |

```
qfgets(buf, sizeof(buf), handle);
return qstrdup(buf);
```

|

| `rename` | `return rename(oldname, newname);` |
| --- | --- |
| `rotate_left` | `return rotate_left(value, count, nbits, offset);` |
| `savefile` | `base2file(handle, pos, ea, ea + size);` |
| `set_start_cs` | `//macro, see SetLongPrm` |
| `set_start_ip` | `//macro, see SetLongPrm` |
| `setattr` | `return VarSetAttr(self, attr, value) == 0;` |
| `sizeof` |

```
type_t *t = internal_type_from_idc_typeinfo(type);
return get_type_size(idati, t);
```

|

| `sprintf` |
| --- |

```
qstring buf;
buf.sprnt(format, ...);
return qstrdup(buf.c_str());
```

|

| `strfill` |
| --- |

```
qstring s;
s.resize(len + 1, &chr);
return new qstring(s);
```

|

| `strlen` | `return strlen(str);` |
| --- | --- |
| `strstr` | `return strstr(str, substr);` |
| `substr` | `Calls internal slice routine` |
| `trim` | `return new qstring(string.c_str());` |
| `unlink` | `return _unlink(filename);` |
| `writelong` | `fwritebytes(handle, &dword, 4, mostfirst);` |
| `writeshort` | `fwritebytes(handle, &word, 2, mostfirst);` |
| `writestr` | `qfputs(str, handle);` |
| `xtol` | `return strtoul(str, NULL, 16);` |
