https://www.nginx.com/blog/optimizing-web-servers-for-high-throughput-and-low-latency/#1445SysctltuningDosandDonts

14:45 Sysctl Tuning Do’s and Don’ts

![123](https://cdn-1.wp.nginx.com/wp-content/uploads/2017/11/Ivanov-conf2017-slide27_Sysctl-Tuning.png)


You can’t actually do any network optimization talk without mentioning sysctl. Here are some do’s and don’ts.

![jpg](https://cdn-1.wp.nginx.com/wp-content/uploads/2017/11/Ivanov-conf2017-slide28_Sysctl-Tuning-Donts.png)

Let’s start with stuff you shouldn’t do:

Don’t enable TIME‑WAIT recycle. It was already broken for NAT [Network Address Translation] users; it will be even more broken in newer kernels. And it’ll be removed, so don’t use it.
Don’t disable timestamps unless you know what you’re doing. It has very non‑obvious implications, for example for SYN cookies. And there are many such implications, so unless you know what you’re doing – unless you’ve read the code – don’t disable them.

![jpg](https://cdn-1.wp.nginx.com/wp-content/uploads/2017/11/Ivanov-conf2017-slide29_Sysctl-Tuning-Dos.png)

Stuff you should do, on the other hand:

You should probably disable net.ipv4.tcp_slow_start_after_idle. There’s no reason to keep it on.
You should enable net.ipv4.tcp_mtu_probing just because the Internet is full of ICMP [Internet Control Message Protocol] black holes.
Bump up read and write memory [net.ipv4.tcp_{r,w}mem]. Not too much, though, just match it to BDP [the Bandwidth Delay Product]. Otherwise, you may get built‑in kernel locking[?].
