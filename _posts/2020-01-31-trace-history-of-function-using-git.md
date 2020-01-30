---
title: "Trace the history of a function using git"
tags:
  - programming
  - git
---

<p class="notice--primary">tl;dr: <code>git log -L :funcname:file</code></p>

Recently I had the need to know which commit in a [git](https://git-scm.org) repository introduced a certain change to some function's logic.

I already knew how to see the change history of an entire file using `git log --patch -- <file>`.
However, there was a lot of history and the file was big, so `git log` generated lots of output I did not care for.

I wondered if it was possible to only see the change history of some function in a file instead of the entire file's.
Guess what, it is possible: [`git log -L :funcname:file`][Lparameter].

### Example
Let us look at a simple example. Consider we have the following source code in the file `example.c`:
```c
void a_function(double a_parameter) {
    // I wish I was special
    // So very special
    // But I'm a function
}

int some_function(int some_parameter) {
    // Do stuff to the parameters
}

void our_function() {
    // I'm your function
    // Here I am
    // This is me
}

double some_other_function() {
    // I am also here
}
```
Now consider that we made some commits with changes to `example.c`.
For example, here is our hypothetical history of the changes to `example.c` in chronological order, courtesy of `git log --patch --reverse -- example.c`:[^1]
```diff
$ git log --patch --reverse -- example.c
commit dc74fc966e1abaf5ff9b5c2763b59f82b917f190
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:03 2038 +0000

    Add example.c

diff --git a/example.c b/example.c
new file mode 100644
index 0000000..d44971c
--- /dev/null
+++ b/example.c
@@ -0,0 +1,19 @@
+void a_function(double a_parameter) {
+    // I wish I was special
+    // So very special
+    // But I'm a function
+}
+
+int some_function(int some_parameter) {
+    // Do stuff to the parameters
+}
+
+void our_function() {
+    // I'm your function
+    // Here I am
+    // This is me
+}
+
+double some_other_function() {
+    // I am also here
+}

commit 3851c5f09f8c73859dc0b5feed90677e58e61f97
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:04 2038 +0000

    Change our_function

diff --git a/example.c b/example.c
index d44971c..e14a2cf 100644
--- a/example.c
+++ b/example.c
@@ -10,8 +10,7 @@ int some_function(int some_parameter) {
 
 void our_function() {
     // I'm your function
-    // Here I am
-    // This is me
+    // And I'm going through changes
 }
 
 double some_other_function() {

commit f3373bc8b9bf64c0af4eb6349a4e9789a5721389
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:05 2038 +0000

    Change a_function

diff --git a/example.c b/example.c
index e14a2cf..3d16bc9 100644
--- a/example.c
+++ b/example.c
@@ -2,6 +2,8 @@ void a_function(double a_parameter) {
     // I wish I was special
     // So very special
     // But I'm a function
+    // This guy is a weirdo
+    // What the hell am I doing here
 }
 
 int some_function(int some_parameter) {

commit d82c3535871fd47ee7bcb0b0f6ff9f5dd30fa6fb
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:06 2038 +0000

    Change some_function and some_other_function

diff --git a/example.c b/example.c
index 3d16bc9..dac2191 100644
--- a/example.c
+++ b/example.c
@@ -8,6 +8,7 @@ void a_function(double a_parameter) {
 
 int some_function(int some_parameter) {
     // Do stuff to the parameters
+    // Return the stuff done to the parameters
 }
 
 void our_function() {
@@ -17,4 +18,5 @@ void our_function() {
 
 double some_other_function() {
     // I am also here
+    // Thanks for acknowledging me
 }

commit dde094c8f47c4f7bfa7176e25653120178aaad51
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:07 2038 +0000

    Change a_function and our_function

diff --git a/example.c b/example.c
index dac2191..f437330 100644
--- a/example.c
+++ b/example.c
@@ -4,6 +4,7 @@ void a_function(double a_parameter) {
     // But I'm a function
     // This guy is a weirdo
     // What the hell am I doing here
+    // I don't belong here
 }
 
 int some_function(int some_parameter) {
@@ -14,6 +15,7 @@ int some_function(int some_parameter) {
 void our_function() {
     // I'm your function
     // And I'm going through changes
+    // So long and thanks for all the changes
 }
 
 double some_other_function() {
```
Now let us image we are interested in seeing the evolution of `our_function()`.
Some commits change `our_function()` and some do not.
This is a lot of output we do not care about.
Here comes the `-L` parameter to the rescue, `git log -L :our_function:example.c --reverse`:
```diff
$ git log -L :our_function:example.c --reverse
commit dc74fc966e1abaf5ff9b5c2763b59f82b917f190
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:03 2038 +0000

    Add example.c

diff --git a/example.c b/example.c
--- /dev/null
+++ b/example.c
@@ -0,0 +11,6 @@
+void our_function() {
+    // I'm your function
+    // Here I am
+    // This is me
+}
+

commit 3851c5f09f8c73859dc0b5feed90677e58e61f97
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:04 2038 +0000

    Change our_function

diff --git a/example.c b/example.c
--- a/example.c
+++ b/example.c
@@ -11,6 +11,5 @@
 void our_function() {
     // I'm your function
-    // Here I am
-    // This is me
+    // And I'm going through changes
 }
 

commit dde094c8f47c4f7bfa7176e25653120178aaad51
Author: John Doe <john.doe@domain.com>
Date:   Tue Jan 19 03:14:07 2038 +0000

    Change a_function and our_function

diff --git a/example.c b/example.c
--- a/example.c
+++ b/example.c
@@ -14,5 +15,6 @@
 void our_function() {
     // I'm your function
     // And I'm going through changes
+    // So long and thanks for all the changes
 }
``` 
Boom! Thanks git. :-)

[^1]: `--patch` shows the patch in the log, and `--reverse` shows the log in chronological order.
[Lparameter]: https://www.git-scm.com/docs/git-log#Documentation/git-log.txt--Lltfuncnamegtltfilegt
