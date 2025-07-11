# PLOOC (Protected Low-overhead Object-Oriented Programming with ANSI-C) v4.6.5

## Introduction

---

Protected Low-overhead Object-Oriented Programming with ANSI-C, known as PLOOC ['plu:k], is a set of well-polished C macro templates that:

- Provide __protection for private__ class members.

> NOTE: This protection can be disabled by defining the macro `__OOC_DEBUG__` to facilitate debugging.

- Support __protected__ members.
- Support __multiple inheritance__.
- Support interface implementation.
- Support strict type checking/validation in certain compilers, such as IAR with multi-file compilation enabled.
- Comply with __ANSI-C99__.
  - ANSI-C90 is also supported, but private member protection is disabled.
- Support **Overload**.
  - Requires C11 or `_Generic`.
- Offer low overhead.
> NOTE: The overhead is nearly ZERO. The template fully utilizes ANSI-C's enforced compilation rules to deliver object-oriented features with minimal necessary cost.

        - Suitable for both bare-metal and RTOS environments.
        - Suitable for both 8-bit and 32-bit MCUs.

### What Makes PLOOC Different from Other OOCs?

The concept of Object-Oriented C (OOC) is not new. There are numerous libraries, SDKs, and templates that extend ANSI-C with object-oriented programming features. Although PLOOC emphasizes its low overhead in both code size and performance, many macro-based OOC solutions also offer low overhead. PLOOC does not force you to use heap or pool memory management, nor does it provide garbage collection (GC) features, leaving these options to the user. This makes it suitable even for 8-bit systems. While some might see this as a drawback, it is a deliberate design choice. I won't argue about this.

**So, what really sets PLOOC apart from the others? Is it simply another reinvented wheel?**

The answer is NO.

PLOOC introduces a unique feature that most other solutions lack: it ensures that a class's private members are truly private and protected. This means that users outside of the class source code are prevented from accessing these private members. Instead, they see only a solid block of memory, masked as a byte array. Since classes in C are mimicked by structures, PLOOC implements classes using a masked structure. As expected, only the class source code can access the private members, and only the class source code of a derived class can access the protected members of the base class. Public members, however, are accessible to everyone.

How does this work? You might have guessed it from the term "masked structure." Essentially, it's a type-cheating trick in the header files. 

This trick works well until it encounters a strict type-checking compiler. The most famous (or notorious) example is IAR, especially when multi-file compilation mode is enabled. No type-cheating trick can survive the rigorous checks of IAR's multi-file compilation mode.

    //! The original structure in the class source code
    struct byte_queue_t {
        uint8_t   *pchBuffer;
        uint16_t  hwBufferSize;
        uint16_t  hwHead;
        uint16_t  hwTail;
        uint16_t  hwCount;
    };
    
    //! The masked structure: the class byte_queue_t in the header file
    typedef struct byte_queue_t {
        uint8_t chMask [sizeof(struct {
            uint8_t   *pchBuffer;
            uint16_t  hwBufferSize;
            uint16_t  hwHead;
            uint16_t  hwTail;
            uint16_t  hwCount;
        })];
    } byte_queue_t;

To make this work, you must ensure that the class source code does not include its own interface header file. You can even go further if you're serious about the content:

    //! The masked structure: the class byte_queue_t in the header file
    typedef struct byte_queue_t {
        uint8_t chMask [sizeof(struct {
            uint32_t        : 32;
            uint16_t        : 16;
            uint16_t        : 16;
            uint16_t        : 16;
            uint16_t        : 16;
        })];
    } byte_queue_t;

> NOTE: The above code snippets are provided to illustrate the underlying scheme but are not practical, as the original structure's alignment is missing when creating the mask array. To solve this, you need to extract the alignment information using the `__alignof__()` operator and set the mask array's alignment attribute accordingly using `__attribute__((aligned()))`.

PLOOC provides the "private-protection" feature with a different scheme, rather than relying on type-cheating. This allows it to support almost all C compilers with C99 features enabled. As the author, I must confess that it took considerable time to figure out how to deal with strict type-checking, and the initial scheme was both ugly and counterintuitive. Thanks to the inspiring contributions of Simon Qian, it took another three months to refine PLOOC into something elegant and simple. Henry Long's support was also crucial.

I hope you enjoy this unique approach to the challenges of object-oriented programming in C.

If you have any questions or suggestions, please feel free to let us know.

## Update Log
---
- \[07/06/2025\] Improve compatibility, version 4.6.5
  - Undefine some macros to avoid warnings
  
- \[08/25/2024\] Fix class template, version 4.6.4
  - Updated Readme

  - Add `__plooc_malloc_align()` and `__plooc_free`

  - Add `private_method()`, `protected_method()` and `public_method()`

  - Remove the dependency on the GNU extensions

  - Other minor changes.

- \[11/02/2022\] Fix class template, version 4.6.3

- \[12/05/2022\] Improve compatibility with the latest C++ language, version 4.6.2

- \[02/01/2022\] Add helper macros for heap, version 4.6.1
    - Add `__new_class()` and `__free_class()` for using malloc and free together with constructors and destructors, i.e. ***xxxx_init()*** and ***xxxx_depose()***.
    - Update example project
    - Add CI to github. 

- \[30/12/2021\] Improved CMSIS-Pac, version 4.6.0
    - Added example project to CMSIS-Pack
    - Added code templates for creating new base classes and derived classes.
    - Other minor updates
- \[29/12/2021\] Add CMSIS-Pack, version 4.5.9
    - Suppressed warnings when c99 is used for the LLVM and GCC. 
    - Fixed an unaligned access issue in the example
    - Added CMSIS-Pack 
- \[28/11/2020\] Minor update, version 4.5.7
    - Fixed a typo in plooc.h 
    - Applied unified capitalisation in macros, i.e. if the macro is uppercase, the parameters are in uppercase, if the macro is lowercase, the parameters are in lowercase.  
    - Added warning to indicate that black box template is incompatible with other templates and should be used alone or under special rule when mixed with other templates.
- \[05/08/2020\] Add \_\_PLOOC\_CLASS\_IMPLEMENT\_\_ and \_\_PLOOC\_CLASS\_INHERIT\_\_ version 4.5.6
    - used \_\_xxxxx\_\_ as emphasis because \_\_xxxxx usually means "internal"
    - The original \_\_PLOOC\_CLASS\_IMPLEMENT and \_\_PLOOC\_CLASS\_INHERIT are deprecated and will be kept for a while before completely removed. 
- \[18/05/2020\] Introduce both short- and long- style of macro, version 4.5.5
    - dcl_xxxxx/declare_xxxxx
    - def_xxxx/define_xxxxx; end_def_xxxx/end_define_xxxx
- \[16/05/2020\] Minor Update, version 4.5.4a
    - Introduced \_\_OOC\_CPP\_\_ to replace \_\_OOC\_DEBUG\_\_ when you want to mix C source code with C++ source code. Please put it in the project global configuration. 
- \[11/05/2020\] Minor Update, version 4.5.4
    - Made it possible to use PLOOC based C source code in C++ project
        - Please make sure \_\_OOC\_DEBUG\_\_ is defined in the project 
- \[15/04/2020\] Update __PLOOC_EVAL, version 4.5.3
    - Increased the range of number of arguments, from 1~8 to 0~16.
- [19/02/2020] Minor update to enable RAM footprint optimisation, version 4.52
    - Introduced macro PLOOC_CFG_REMOVE_MEMORY_LAYOUT_BOUNDARY___USE_WITH_CAUTION which removes structure layout boundaries for PLOOC_VISIBLE. It can save RAM when certain condition is met and \_\_OOC\_DEBUG\_\_ is defined. Please use it with caution as it will cause different memory layouts when \_\_OOC\_DEBUG\_\_ is not defined. 
- \[21/01/2020\] Misc update for C90, version 4.51
- \[09/06/2019\] Added support for C89/90, version 4.50
    - Added full support for overload \(require C11\)
- \[09/05/2019\] Added support for C89/90, version 4.40
    - When C89/90 is enforced, \_\_OOC_DEBUG\_\_ should always be defined. 
    - The protection for private and protected members is turned off.
- \[08/15/2019\] Updated plooc_class_strict.h to use more soften syntax, version 4.31
    - Users now can use arbitrary order for public_member, private_member and protected_member.
    - The separator "," can be ignored. 
    - Simplified the plooc_class_strict.h template. Some common macros are moved to plooc_class.h, which will be shared by other template later. 
- \[08/14/2019\] Introduced support for limited support for overload, version 4.30
    - Used can use macro \_\_PLOOC_EVAL() to select the right API which has the corresponding number of parameters. 
- \[07/26/2019\] Syntax update, version 4.21
    - Modified plooc_class_black_box.h to use unified syntax as other templates.
    - Added extern_class and end_extern_class to all templates
- \[07/24/2019\] Added new ooc class template, version 4.20
    - Added plooc_class_black_box.h. This template is used for creating true-black-box module. It only support "private" and "public" but no "protected".  
- \[07/12/2019\] Minor Update, version 4.13
    - Added "\_\_OOC_RELEASE\_\_". The struct requires protection only at development stage. For private properties, setters and getters are provided for controlling the access. It is possible to remove masks and allow private members observable in release stage, during this stage, the setters and getters can be changed from API functions to macros. By doing so, the code size can be smaller.
- \[05/30/2019\] Minor Update, version 4.12
    - removed "this", "target" and "base" to prevent naming pollution.
    - removed PLOOC_ALIGN from top-level class definition to prevent inconsistent compiler interpretation towards this alignment decoration. 
- \[05/02/2019\] Efficiency improve, version 4.11
    - Used \_\_alignof\_\_ to improve the code efficiency when dealing with masked structure
    - Used PLOOC_INVISIABLE and PLOOC_VISIBLE in both simple and strict version
    - Simplified the structure
    - Improved capability between IAR and armclang (LLVM)
- \[05/01/2019\] Compatibility Improving, version 4.04
    - Added PLOOC_PACKED and PLOOC_ALIGN to add alignment support
    - Used uint_fast8_t to replace uint8_t to use target machine implied alignment.
- \[04/20/2019\] Upload PLOOC to github, version 4.03
    - Added default class alignment control
    - updated examples and readme
- \[04/17/2019\] Uploaded PLOOC to github, version 4.01
    - Added method definition which support private method, protected method and public method
    - Added readme and example byte_queue


## License
---
The PLOOC library was written by GorgonMeducer(王卓然）<embedded_zhuoran@hotmail.com> and Simon Qian（钱晓晨）<https://github.com/versaloon> with support from Henry Long <henry_long@163.com>.

The PLOOC library is released under an open source license Apache 2.0 that allows both commercial and non-commercial use without restrictions. The only requirement is that credits is given in the source code and in the documentation for your product.

The full license text follows:

	/*****************************************************************************
	 *   Copyright(C)2009-2019 by GorgonMeducer<embedded_zhuoran@hotmail.com>    *
	 *                       and  SimonQian<simonqian@simonqian.com>             *
	 *         with support from  HenryLong<henry_long@163.com>                  *
	 *                                                                           *
	 *  Licensed under the Apache License, Version 2.0 (the "License");          *
	 *  you may not use this file except in compliance with the License.         *
	 *  You may obtain a copy of the License at                                  *
	 *                                                                           *
	 *     http://www.apache.org/licenses/LICENSE-2.0                            *
	 *                                                                           *
	 *  Unless required by applicable law or agreed to in writing, software      *
	 *  distributed under the License is distributed on an "AS IS" BASIS,        *
	 *  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. *
	 *  See the License for the specific language governing permissions and      *
	 *  limitations under the License.                                           *
	 *                                                                           *
	 ****************************************************************************/


## Contribution
---
### Template
| module | Contrinutor |
| ------ | ------ |
| plooc.h | GorgonMeducer |
| plooc_class.h | GorgonMeducer, Simon Qian |
| plooc_class_strict.h | GorgonMeducer |
| plooc_class_back_box.h | GorgonMeducer |
| plooc_class_simple.h | Simon Qian |
| plooc_class_simple_c90.h | GorgonMeducer |


### Examples
| module | Contrinutor |
| ------ | ------ |
| How to define a class | GorgonMeducer |
| How to access protected members | GorgonMeducer |
| How to implement Polymorphism | GorgonMeducer |

## Applications / Projects which claim to use PLOOC
---
- [VSF][https://github.com/vsfteam/vsf]
- [GMSI][https://github.com/GorgonMeducer/Generic_MCU_Software_Infrastructure]
- [mqttclient][https://github.com/jiejieTop/mqttclient]

## How to Use
---
### Examples for PLOOC
#### Introduction
In order to show how PLOOC is easy and simple to use, examples are provided to demonstrate different aspects of the new OOPC method. Currently, the available examples are:

- byte_queue
- enhanced_byte_queue

More examples will be added later...

- ### [Example 1: How to define a class](https://github.com/GorgonMeducer/PLOOC/tree/master/example/byte_queue)

  This example shows

  - How to define a class
    - How to add private member
    - How to add protected member
  - How to access class members
  - How to define user friendly interface

  ### [Example 2: How to access protected members](https://github.com/GorgonMeducer/PLOOC/tree/master/example/enhanced_byte_queue)

  - How to inherit from a base class
    - How to access protected members which are inherited from base
  - How to inherit a interface
  - How to override base methods

  ### [Example 3: How to implement Overload ](https://github.com/GorgonMeducer/PLOOC/tree/master/example/trace)

  - How to implement overload using PLOOC

  - Require C11 support

```
LOG_OUT("\r\n-[Demo of overload]------------------------------\r\n");
LOG_OUT((uint32_t) 0x12345678);
LOG_OUT("\r\n");
LOG_OUT(0x12345678);
LOG_OUT("\r\n");
LOG_OUT("PI is ");
LOG_OUT(3.1415926f);
LOG_OUT("\r\n");

LOG_OUT("\r\nShow BYTE Array:\r\n");
LOG_OUT((uint8_t *)main, 100);

LOG_OUT("\r\nShow Half-WORD Array:\r\n");
LOG_OUT((uint16_t *)main, 100/sizeof(uint16_t));

LOG_OUT("\r\nShow WORD Array:\r\n");
LOG_OUT((uint32_t *)main, 100/sizeof(uint32_t));
```



![example3](./example/picture/example3.png?raw=true)

  

