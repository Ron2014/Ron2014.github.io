---
layout: page 
title: About

---

## Base Infomation

* Chinese Name: Ruilong Ji
* English Name: Ron Cybeureka
* Brithplace: Huangshi, Hubei, China
* Cencus Register: Guangzhou, Guangdong, China
* Marital Status: Single
* Work Mailbox: dev_ron@163.com
* Blog: https://pensieve.cybeureka.com/
* GitHub: https://github.com/Ron2014
* School: Software Engineering, South China Agricultural University

## Job Intention

* Game Server Development Engineer
* Gameplay Developer

## Career History (From Now to the beginning)

### Longtu Game

Time: 2021.1.4 - now

Title: Senior Game Server Development Engineer

Project: Contra Battle Royale for mobile

Game Category: Frist Person Shooter base on Room/Matching

Tools:

  1. SVN
  2. Visual Studio Code
  3. XMind
  4. TAPD
  5. Windows Terminal
  6. CentOS 7.7

Development Environment:

  1. MySQL
  2. Redis
  3. C++
  4. Python
  5. Shell

Job Responsibilities:

1. Design and Develop game server program, including architecture and logic, schema of database, IPC protocol, etc.
2. Design and optimize the architecture of game server to guarantee the high concurrency which can reach 10W peak concurrent users.
3. Based on the requirements from Game Designer, develop, test and maintain the logic codes, solve the techical problem as a team leader.
4. Finish the development tasks and commit the new version of project on time based on the process of plan.

Job Requirements:

1. Undergraduate or above, major in Computer Science, Software Engineering etc.
2. Excel in C++ Language, participated in the core development of at least one mobilegame/webgame/pc-online-game.
3. Familiar with common database projects, e.g. mysql, redis. Excel in Developing in Linux OS. Excel in Data Structure and Algorithm. Excel in programming with sockets, multi-progress and multi-thread.
4. Familiar with the theory of designing a high throughput, high concurrency, high performance distributed system.
5. Technical in communication and cooperation, high sense of responsibility and professional dedication.
6. Strong anti-pressure ability, which can discover and make move to solve the problems of system individually and efficiently.

Experiences:

1. Based on the single point architecture server system, developing iteratively to build a multi-point distributed server system which make no sense of game zone for players.
2. Based on the new server architecture, implemented a matching system for single player requests and team requests.
3. Based on the new server architecture, integrated Redis database to cache and share datas in multi-process environment, made the solution to update and sync datas to guarantee the consistency.
4. Based on the new server architecture, extended the function of RPC, deal with different message pathing to different type of server system, made the solution of IPC in multi-process environment.
5. Make plan and assign tasks to transplant the exist systems in the single point architecture server system to the new one, mapping the plan, packaging functional codes and developing tools to make the tasks easy and efficiently.
6. For the communication protocol between client and servers, write scripts to check the consistency, generate common logic codes, generate the test cases to make development high-efficient and get rid of some runtime errors.
7. Document for system deploy, write scripts for system deploy in one click.
8. Collect the problems and deficiencies of the new system, research and document, make workshops on these subjects, make moves to solve them and report the progressy in real time.
9. Technical interviewer of our project. Responsible for staff appraisal.

### Zhiwan Game

Time: 2020.7.1 - 2020.12.18

Title: Senior Game Server Development Engineer & IT Engineer

Project: 魂之歌/契约战歌 for mobile

Game Category: 2D Cards Game, Turn-based

Tools:

  1. SVN
  2. Visual Studio Code
  3. XMind
  4. Redmine
  5. SecureCRT
  6. Windows Terminal
  7. CentOS 7.0

Development Environment:

  1. MySQL
  2. MongoDB
  3. C
  4. Lua
  5. Python
  6. Shell

Job Responsibilities:

1. Guarantee the stability of server functions, while fixing bugs and defects.
2. Maintain and extend the economic system, battle system, etc.
3. Based on the requirements of system design, make goals, milestones and schedule, estimate time to move the progress on.
4. Technical interviewer of game server developer, appraise for newbie.
5. Instruct the development of game server management website system, which for collecting BI logs, executing GM commands and generating digital redeem codes.
6. Daily operate, maintain and monitor the systems instances on the tencent cloud service, deploy the server systems for online test or official operation.

Experiences:

1. Code reviewed for being familiar with design of systems, sometimes retype and format to get higher readablility and expansilility.
2. Scripted tools to debug and located the bugs or performance bottle-necks.
3. Made robot programs to simulate the real player login, logout actions to get the peak concurrent users of our system. Optimized the work flow of login, logout, secure the shutdown of server system.
4. Scripted to simulate battle request, assist the data designer to calculate the datas of pvp gameplay.
5. Based on the flame graph tool and login/logout/battle request protocol, located the hotspot in functions, improved the ablity of response concurrent requests.
6. Built the quene mechanics for reentrant coroutine, solved the problems among async calling, implement singleton.
7. Text parsing protocols by sproto, made protocols automatic unit test environment.
8. Solution for config files hotfix, memory shared, code cached.
9. Scripted shell and extended debug console tool to monitor the memory and cpu of processes.
10. Based on object relationship model, rewrote the logic of player login, initialization, data loading, data updating. Used dirty mark in property sync to client or database.
11. Brand new of battle system, new and configurable skill target select system for different and complicate requirements, new work flow of battle rules, new skill system with finite-state machine and passive skill, new buff system with cached data and dirty mark in property calculation system to improve efficiency about 100 times, and more and more maintainability and expandability.

### Yixin Game

Time: 2019.11.18 - 2020.03.27

Title: C++ Game Server Development Engineer

Project: Pre-research project

Tools:

  1. SVN
  2. Visual Studio Code
  3. Visual Studio
  4. XMind
  5. Ubuntu
  6. WSL2
  7. Docker
  8. FIPS
  9. CMake

Development Environment:

  1. leveldb
  2. C++
  3. Lua
  4. Python
  5. TypeScript
  6. TypeScriptToLua

Job Responsibilities:

1. Confirm the requirements by communications with designers and artists, give the program solution.
2. Code the system in server with high quality and efficiently, fix the bugs.
3. assist the programmer manager to profile server system architecture design.

Experiences:

1. Intergrated lua-cjson, curl library.
2. Lua scripts encryption. Used python to parsing lua scripts, finish the function by change the code of virtual machine. Just in 5 steps:
   1. Shuffle the 47 opcode enums.
   2. For opcode which store by 6 bits in memory, define 17 Non-sense opcodes which called NOP command among the real enums, insert them when virtual machine parsing lua script and building proto object by percent which can be configured in python scripts.
   3. Because the NOP opcodes, relative address jump in lua bytecode which call iAsBx mode need change the relative address parameters. I rewrote the lua function parsing way to do it.
   4. To make the NOP opcodes more obfuscated, I counted the bytecode information of 300 lua scripts, got the opcodes' mode and region of parameters. Random parameters in common values for the NOP opcodes.
   5. For the string encryption, random generate pass phrase for 2^N bytes long, XOR the source.
3. Design the battle system with CTO.

### Netease

Time: 2015.03.05 - 2019.06.04

Title: Senior Game Research and Development Engineer

Job Responsibilities:

1. Major in Computers.
2. Familiar with C/C++, good with mathematics, data struction, algorithm and common design pattern.
3. Familiar with one script language.
4. Familiar with programming for multi-thread and network.
5. Familiar with PC / mobile development work flow.
6. Familiar with performance and memory optimizing.
7. High responsibility and good at communication.

Projects:

#### 1. 猫咪争霸/无限战争

Game Category: 2D Real Time Strategy Mobile Game

Time: 2015.03.05 - 2017.03.07

Tool:

  1. SVN
  2. Sublime Text
  3. Visual Studio

Development Environment:

  1. MySQL
  2. MongoDB
  3. C++
  4. Lua
  5. Python
  6. Cocos2d-lua
  7. PyQt
  8. Self-innovate server engine
