---
title: macOS下的bits/stdc++.h
date: 2020-03-03 19:02:55
abstract: 如果你对这个头文件名称比较陌生，便不必点开。
tags: ["C/C++", "macOS"]
categories: "Blog"
---

&#160; &#160; &#160; &#160; 如题，macOS默认是没有\<bits/stdc++.h\>这个头文件的。原因的话，应该是Mac默认使用的编译器是clang，而非GNU提供的gcc、g++。

&#160; &#160; &#160; &#160; 网上关于如何在macOS下使用\<bits/stdc++.h\>的讨论不少，其中大部分是直接从根本上解决问题：直接卸载clang，去GNU下载gcc、g++并安装。

&#160; &#160; &#160; &#160; 但是，如果你也像我一样对clang还比较留恋、且比较懒的话(或许我该把主要原因写在前面？)，可以试试如下方法：

&#160; &#160; &#160; &#160; 切换至如下目录：

&#160; &#160; &#160; &#160; `/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include/`

&#160; &#160; &#160; &#160; 创建目录`bits`(你应该已经知道了下一步该怎么弄)。

&#160; &#160; &#160; &#160; 在目录`bits`下，创建文件`stdc++.h`。当然，这里可能需要你的权限，给它便是。

&#160; &#160; &#160; &#160; 让你的`stdc++.h`文件长成这个样子：

```cpp

// C++ includes used for precompiling -*- C++ -*-

// Copyright (C) 2003-2014 Free Software Foundation, Inc.
//
// This file is part of the GNU ISO C++ Library.  This library is free
// software; you can redistribute it and/or modify it under the
// terms of the GNU General Public License as published by the
// Free Software Foundation; either version 3, or (at your option)
// any later version.

// This library is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.

// Under Section 7 of GPL version 3, you are granted additional
// permissions described in the GCC Runtime Library Exception, version
// 3.1, as published by the Free Software Foundation.

// You should have received a copy of the GNU General Public License and
// a copy of the GCC Runtime Library Exception along with this program;
// see the files COPYING3 and COPYING.RUNTIME respectively.  If not, see
// <http://www.gnu.org/licenses/>.

/** @file stdc++.h
 *  This is an implementation file for a precompiled header.
 */

// 17.4.1.2 Headers

// C
#ifndef _GLIBCXX_NO_ASSERT
#include <cassert>
#endif
#include <cctype>
#include <cerrno>
#include <cfloat>
#include <ciso646>
#include <climits>
#include <clocale>
#include <cmath>
#include <csetjmp>
#include <csignal>
#include <cstdarg>
#include <cstddef>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <ctime>

#if __cplusplus >= 201103L
#include <ccomplex>
#include <cfenv>
#include <cinttypes>
#include <cstdbool>
#include <cstdint>
#include <ctgmath>
#include <cwchar>
#include <cwctype>
#endif

// C++
#include <algorithm>
#include <bitset>
#include <complex>
#include <deque>
#include <exception>
#include <fstream>
#include <functional>
#include <iomanip>
#include <ios>
#include <iosfwd>
#include <iostream>
#include <istream>
#include <iterator>
#include <limits>
#include <list>
#include <locale>
#include <map>
#include <memory>
#include <new>
#include <numeric>
#include <ostream>
#include <queue>
#include <set>
#include <sstream>
#include <stack>
#include <stdexcept>
#include <streambuf>
#include <string>
#include <typeinfo>
#include <utility>
#include <valarray>
#include <vector>

#if __cplusplus >= 201103L
#include <array>
#include <atomic>
#include <chrono>
#include <condition_variable>
#include <forward_list>
#include <future>
#include <initializer_list>
#include <mutex>
#include <random>
#include <ratio>
#include <regex>
#include <scoped_allocator>
#include <system_error>
#include <thread>
#include <tuple>
#include <typeindex>
#include <type_traits>
#include <unordered_map>
#include <unordered_set>
#endif

```

&#160; &#160; &#160; &#160; 以上，即可。
