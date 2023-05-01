# ECE650 Assignment5 RootKit
## Checkpoint1: Hijacking openat()
### First Step: Substitute function pointer of the original openat() to the function pointer of your malicious openat(), but simply invoke the original openat()
### Second Step: Print out the pathname that is passed to openat()
Check the manual to see the function signature of openat()
```
man openat
```

The signature in the code is
```
asmlinkage int sneaky_sys_openat(struct pt_regs * regs)
```
How to get the parameters passed in?

To get the first parameter, use
```
regs->di
```
To get the second parameter, use
```
regs->si
```
To get the third parameter, use
```
regs->dx
```
To get the fourth parameter, use
```
regs->r10
```

How to print out the information?
```
printk(KERN_INFO "Hello!");
```
Where to see the printed information?
```
sudo cat /var/log/syslog
```
One thing to pay attention: parameter pathname is a const char *, which is a pointer to **user-space memory**. However, our kernel module will be run in **kernel-space**, which is a seperate memory space from user. Therefore, the user-space memory region needs to be copied to kernel-space before further processing.

How to copy memory from user-space to kernel-space?
```
__copy_from_user(<a kernel-space pointer>, <a user-space pointer>, <number of bytes to copy>)
```
When you finish this step, you should be able to see "/etc/passwd" in /var/log/syslog when you execute
```
cat /etc/passwd
```
### Third Step: Substitute pathname and pass it to the original openat()
One thing to pay attention, the substituted pathname pointer should also be a user-space pointer. Therefore, we need to copy the substituted pathname to the user-space before passing it to the original openat().

How to copy memory from kernel-space to user-space?
```
__copy_to_user(<a user-space pointer>, <a kernel-space pointer>, <number of bytes to copy>)

```

When you finish the above steps, you should not be able to see the modication to "/etc/passwd", because the parameter pathname of openat() is secretly replaced to "/tmp/passwd"!
