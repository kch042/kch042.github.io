---
title: 2024 SWE New Grad 求職紀錄
cover: /img/stretch.png
mathjax: true
categories: Career
tags: 
    - NCG
    - Nvidia
    - Google
    - Synology
    - TSMC
abbrlink: 324802
date: 2023-12-06 00:00:00
updated: 2024-04-22 00:00:00
---

這篇紀錄了我初期找工作的一些經過，後續還有與其他公司(e.g. ByteDance)面試，但因為後來太忙就沒時間記錄下來。

## TL;DR
- Offer: NV, Google SWE Intern
- Reject: Synology
- No outcome: TSMC
- No interview: Synopsys, Google

## Nvidia TensorRT (RDSS Intern)

### Timing

- D: application submitted
- D+3: HR email me about receiving me resume
- D+5: 1st round interview invitation
- D+9: 1st round interview
- D+11: 2nd round interview invitation
- D+15: 2nd round interview

### 一面

感覺面下來人最親切，學到很多

1 person, 1 hour

- Introduction to TensorRT
- Questions
    - C++
        - virtual function
    - Computer Architecture
        - pipelined CPU
        - cache invalidate
    - Perf Analysis
        - how to analyze the perf of addition of two vectors

### 二面

3 people, each one 1 hour respectively


#### 2-1
- Brief intro of myself
- Resume
    - OS
        - how does a system call work
        - why user mode & kernel mode
        - what metadata does kernel have for each process
        - describe the user thread library I implemented
    - Computer Arch
        - describe my RISV-V CPU implementation
        - what's the challenge in this project
        - why pipelined CPU
- C++
    - my experience in C++ projects
    - pointer vs reference
    - can a class constructor be virtual, and why
    - how does virtual work
    - smart pointers
- Algorithmic problem
    - [finding duplicate subtrees](https://leetcode.com/problems/find-duplicate-subtrees/description/)

2-2
- Perf Analysis
    - suppose I have a branch new CPU which allegedly has good performance and great quality of cache. But I don't believe in that! Suppose I have a compiler that can compile code to run on the CPU, write a code to run to measure the cache size and the level of cache.
- Algorithmic Problem
    - add two numbers represented by a linked list

2-3
感覺面下來人第二好的面試官，可惜沒有能寫到bonus QQ

- Brief self intro
- C++ projects experience
- Resume questions
    - OS
        - many projects, where do you get, how to evaluate
    - DL projects
        - LSTM, how do you build it?
        - text summarization, how do you evaluate them?
- HackerRank
    - Sparse Vector implementation
        - 2 constructors
        - operator get method (O(n) -> O(lgn) -> O(1))
        - add two vectors (O(n))
- QA
    - Does TensorRT team require CUDA skills?
        - 現在TensorRT team把大部分CUDA的功能都切給另個team，這個team現在主要是focus在C++，不過有CUDA知識也很好
    - Does TensorRT team need to train/research model?
        - 不會，但是要很了解network架構，這樣才知道哪些部分可以做加速的優化，例如residual norm layer做完才搬回memory



## Synology 台大快速面試

### Timing
- D: resume submitted
- D+7: interview invitation received
- D+26: interview
- D+33: rejection received

### 一面

1hr, 一位工程師 (from Synology cbt2)

- 情境題:
    - 假設我們有一台遠端server可以讀寫一個巨大的文件，也有一個讀寫此server的record（tsv檔案，column分別是`rw`, `offset of the document`, `length read`）。我們今天發現讀的速度非常的慢，有什麼辦法可以解決這個問題？
    - 我的解法：想設計一個cache來減少對遠端讀的request，在record中看，對於每一個read，後面100個read有多少是被包含在我這個read裡，藉此可得到cache hit rate
    - 也有另一種想法：prefetch
    - 可惜時間不夠，也僅僅夠實作分析解法的可行性，來不及實作cache...


## Nvidia CAD DFT (RDSS Intern)

這team是做in-house的eda modules

### Timing
- D: 投履歷
- D+5: Recruiter聯繫 （跟TensorRT的同一位）
- D+7: 一面
- D+27: 二面邀請
- D+32: 二面
- D+34: Next Step (Recruiter致電，得到口頭offer)
- D+40: Offer Letter


### 一面
面試官是台灣人，前10分鐘主要是問我的經歷，focus在我實習的部分，然後研究的部分問了點，還問了一些RDSS的狀況，偏閒聊。

除此之外還有稍微介紹CAD DFT Team的工作內容，除了開發一些CAD module，還有包含些CI/CD的東西

中間開始coding，考了一題OOP (virtual function)，然後一題BST key search (easy)，一題min path of a tree with specified visited nodes (medium)

最後10分鐘有一些QA
- CAD DFT用的C++版本：11，不太會用到很fancy的功能，因為有些古老的tcl script有dependency的問題
- CAD DFT需要什麼樣的人：對背景要求還好，主要看中人的personality，能不能解決問題
- CAD DFT在台灣的人數：team裡大概台北5人，新竹5人。如果算上work very closely的team大概全台70-80人

### 二面
on site面試，第一次到內湖NV辦公室，在西湖站旁。

兩位面試官，各一個小時。

第一位一開始請我先自我介紹，接著問了我碩論的東西以及我在自介裡提到的經歷。問完背景相關問題後，他請我介紹一遍我知道的資料結構與演算法（一個一個把他講出來...），然後他就出了兩題算法題。（第一題是給一棵binary tree，寫一個function確認他是不是min heap。第二題是找出kth large element in an array）。

這兩題學過算法的都不會覺得太難，但是有些caveat需要注意。例如第一題即使parent node都比child node小，要是樹的形狀不對也不算是min heap。寫完兩題後時間就差不多到了，只剩5分鐘跟我閒聊，第二位面試官就到了。

第二位面試官是RDSS轉正的engineer，資歷比較淺的感覺？他先是拿著履歷問了我的背景，例如實習，修課，擅長的程式語言等等。接著考了兩題算法題，難度一題easy，一題medium，很快就解完了XD (他說我表現得很不錯，比一些面正職的還要好？)。剩下蠻多時間閒聊的，就問了些對職缺的問題
- RDSS容易轉正嗎
    - A: 我們team轉正機率非常高，不像有些team時間到就叫人走人了XD
- 這個工作累嗎？會常加班嗎？
    - A: (露出似乎有點無奈的微笑) 我們沒有規定時間上下班，主要是責任制，project都是一陣一陣的，有做完比較重要。事實上我們辦公室的位子其實不夠，所以大家大部分都是WFH，大家一個禮拜只有一天會來，也許之後搬公司會有位子。
- 其他忘記了...

### Next Step

二面兩天後早上接到Recruiter電話得到口頭offer，接著填寫一些文件和約時間體檢（NV贊助，免費）。Recruiter預計兩週左右的時間跑文件的部分。

### Offer Letter
收到offer letter，上面有薪資，一些規定和福利。這邊RDSS沒有RSU，取而代之的是一包sign-on bonus.

## TSMC SWE

### Timing
- D: 收到邀請 
- D+6: 一面

### 一面
跟主管面試，大概一個小時，一開始他還以為我已經畢業了...，後來他說會再問問看能不能走預聘

面試過程問了碩論跟實習相關的東西，主要會問遇到什麼問題，問題的價值在哪，以及提出怎麼樣的solution去解決

後面主要就是聊天，聊聊台積這幾年的擴廠以及數位轉型

### OA (HackerRank)

三題90分鐘，體感上是E, E/M, M，大概30幾分鐘就交卷了


## Nvidia Display Driver (RDSS Intern)

### Timing
- D: 投遞履歷
- D+11: OA
- D+13: 完成OA
- D+16: 一面邀請
- D+18: 一面 (取消)

### OA

120 mins，考OS題目，一些C++實作題目(bit manipulation，macro相關的題目)和debugging題目


## Others

### Synopsys

新思的HR有打電話給我跟我談recruit的流程，她會把履歷給主管看，如果有主管覺得有興趣會再來找我來面試

### Google

投遞NG後沒多久職缺就關閉了，status仍停留在submitted，我猜NG職缺都被intern先消化完了

## Extra: Google 2024 SWE Summer Intern

無聊投了2024 Google SWE Summer Intern，記錄一下面試流程

### Timing
- D: 投履歷 (12/25/2023)
- D+8: 收到信填寫information collection form (01/02/2024)
- D+28: 收到面試時間通知 (01/22/2024)
- D+32: 一面 (01/26/2024)
- D+35: 二面 (01/29/2024)
- D+39: 收到信recruiter幫我送HC (02/02/2024)
- D+59: HC通過，填寫intern project preference的google form (02/22/2024)
- D+119: Offer Talk (04/22/2024)

### 一面 (2024.01.26)
使用Google meet和一個virtual interview editor進行面試，只有一題

題目是這樣，有使用過Google Photo的都應該知道會有一個slide show的功能，用類似IG限時動態的方式呈現一些回憶照片

這邊題目要設計一個簡單的slide show，給定兩個input, `album`和`favorites`，要去設計一個iterator function `GetNextPhoto()`，每次回傳一張照片，同時滿足先把`favorites`裡的照片回傳完，再回傳`ablum`裡剩下non-favorites的照片

我一開始弄錯意思，寫了一個function一次output出排好題目要求順序的照片

後來弄清楚題意後，我的思路是寫一個class，把`favorites`加進去一個set，再去使用pointer去maintain目前調閱過的照片。後來面試官想問能不能優化空間（set需要O(n)的空間），我就想到先把`album`和`favorites`做sorting，如此可以在調閱`album`的時候可以利用two-pointer快速知道目前的照片是否在favorite裡 （可惜來不及implement）

**這邊總結一下Google面試的點和流程**
- 一開始要跟面試官討論題目跟提出合理的assumption，然後才開始寫扣
- 寫扣時要一邊寫一邊跟面試官講解想法
- 寫完扣會要去分析複雜度，以及提出一些edge case去dry run你的扣
- 還有一些coding style的部分要注意，包括但不限於naming和casing的部分
- 接下來會要去討論一些優化，不管是時間或是空間上的
- 面試時間很嚴格就是45分鐘

### 二面 （2024.01.29）
不同於一面，二面是一個外國人，英文面試

這次考兩題，使用英文面試，一開始先寒暄一下，接著進到coding流程，一樣會先描述題目

這次題目不難，有兩題。第一題是給定一個sentence，去數有幾個word。第二題是給定一串聊天記錄，每句的格式是`timestamp speaker sentence`，去數每個speaker講了幾個word。

一樣作答時先問一些問題模糊的地方，確定一些assumption，接著描述算法概念，然後才開始實作。實作時邊寫邊跟面試官講解邏輯的部份，寫完扣後去想一些edge case來dry run一下，最後討論複雜度的部分。

值得一提的是這次面試官比較不在意naming和coding style的部分，主要是看我的程式正確性（雖然我沒能一開始就寫出bug-free的程式碼，但是後面經過一些提示跟想一些edge case有成功寫出面試官覺得ok的扣）

45分鐘結束後有給5分鐘QA，我臨時想不到一些問題，就隨便問了兩個
- Google Taiwan有很多外國人嗎
    - ans: 以他的team全部100多人，大概有10-15%是外國人
- Google最近裁員有影響到台灣嗎
    - ans: 他也不清楚，所以沒辦法告訴我XD

整題來說面試下來感覺還不錯，題目不會太難，面試官也算親切

### Team Match

只要填一個project表單，HM會去挑人，也不會有Fit Talk。最後我去的team是Platform Infrastructure (PI)，根據HR的資訊是做server相關的，接觸到的teams很多