---
title: "Interface Groups"
date: "2010-07-02"
categories:
  - "software"
tags:
  - "group"
  - "hack"
  - "interface"
  - "openbsd"
---

I hack on OpenBSD in my spare time. (Yeah, that infrequently.) Every year, around hackathon time, I find myself hunting through the guts of the Operating System sources, learning a new trick, or two. Being able to learn like this is the main reason I _use_ open source. This is an example of a typical learn-by-doing session.

OpenBSD has a handy mechanism for assigning network interfaces (which may have decidedly unremarkable names like fxp0, ne1, sis3, and so on) into something called _interface groups_.

An interface group is simply a one-to-many mapping---a named bucket for one or more interfaces. This is handy for when you, say, assign fxp2 to be your synchronization interface, via:

    # ifconfig fxp2 group sync

In theory, referring to "sync" in a configuration file somewhere would be sufficient to use the correct interface.

The problem is, not every piece of software knows about interface groups. Adding this support means understanding how to retrieve this mapping from userland.

Let's have a look at the code. There are two places I can think of that do this mapping: ifconfig, and pfctl. Locating their respective sources gives me a hint:

    $ which ifconfig
    /sbin/ifconfig
    $ which pfctl
    /sbin/pfctl

A quick hunt reveals the source files I'm interested in: `/usr/src/sbin/ifconfig/ifconfig.c` and `/usr/src/sbin/pfctl/pfctl_parser.c`

Not surprisingly, the source are similar in both cases. It turns out we can use the We use the SIOCGIFGMEMB ioctl to do this. To see how, we can have a quick peek at the kernel code that handles it.

Looking in `sys/net/if.c`:

```C
case SIOCGIFGMEMB:
        return (if_getgroupmembers(data));
```

So the relevant kernel code is in `if_getgroupmembers()` which can be found in the same file. It starts like this:

```c
/*
 * Stores all members of a group in memory pointed to by data
 */
int
if_getgroupmembers(caddr_t data)
{
    struct ifgroupreq       *ifgr = (struct ifgroupreq *)data;
    struct ifg_group        *ifg;
    struct ifg_member       *ifgm;
    struct ifg_req           ifgrq, *ifgp;
    int                      len, error;

    TAILQ_FOREACH(ifg, &ifg_head, ifg_next)
            if (!strcmp(ifg->ifg_group, ifgr->ifgr_name))
                    break;
    if (ifg == NULL)
            return (ENOENT);
```

If the group name (supplied to the ioctl) is not found, ENOENT is returned (via errno).


```c
if (ifgr->ifgr_len == 0) {
        TAILQ_FOREACH(ifgm, &ifg->ifg_members, ifgm_next)
                ifgr->ifgr_len += sizeof(ifgrq);
        return (0);

}
```

If (`ifgr_len == 0`) in the passed-in structure, we simply return the amount of storage necessary to store all the interface group members.

```C
        len = ifgr->ifgr_len;
        ifgp = ifgr->ifgr_groups;
        TAILQ_FOREACH(ifgm, &ifg->ifg_members, ifgm_next) {
                if (len ifgm_ifp->if_xname,
                    sizeof(ifgrq.ifgrq_member));
               if ((error = copyout((caddr_t)&ifgrq, (caddr_t)ifgp,
                    sizeof(struct ifg_req))))
                        return (error);
                len -= sizeof(ifgrq);
                ifgp++;
        }

        return (0);
}
```

Otherwise, we store as much of the interface data as we can. Note that to parse the data coming out, we have to use a struct ifgroupreq. We can find this definition in one of the system header files: `net/if.h`:

```c
struct ifgroupreq {
    char    ifgr_name[IFNAMSIZ];
    u_int   ifgr_len;
    union {
            char                     ifgru_group[IFNAMSIZ];
            struct  ifg_req         *ifgru_groups;
            struct  ifg_attrib       ifgru_attrib;
    } ifgr_ifgru;
#define ifgr_group      ifgr_ifgru.ifgru_group
#define ifgr_groups     ifgr_ifgru.ifgru_groups
#define ifgr_attrib     ifgr_ifgru.ifgru_attrib
};
```

To understand how to use it, here's a stripped down version of the function used in `pfctl_parser.c`:

```C
void grouplookup(const char *ifa_name)
{
        struct ifg_req          *ifg;
        struct ifgroupreq        ifgr;
        int                      s, len;
        char                     ifgbuf[IFNAMSIZ+1];

        if ((s = socket(AF_INET, SOCK_DGRAM, 0)) == -1)
                err(1, "socket");
        bzero(&ifgr, sizeof(ifgr));
        strlcpy(ifgr.ifgr_name, ifa_name, sizeof(ifgr.ifgr_name));
        if (ioctl(s, SIOCGIFGMEMB, (caddr_t)&ifgr) == -1) {
                close(s);
                return;
        }

        len = ifgr.ifgr_len;

        if ((ifgr.ifgr_groups = calloc(1, len)) == NULL)
                err(1, "calloc");
        if (ioctl(s, SIOCGIFGMEMB, (caddr_t)&ifgr) == -1)
                err(1, "SIOCGIFGMEMB");

        for (ifg = ifgr.ifgr_groups; ifg && len >= sizeof(struct ifg_req);
            ifg++) {
                len -= sizeof(struct ifg_req);
                bzero(&ifgbuf, sizeof(ifgbuf));
                strlcpy((char *)ifgbuf, ifg->ifgrq_member, sizeof(ifgbuf));
                printf("%s ", ifgbuf);
                /* should do ifa_lookup here */
        }
        free(ifgr.ifgr_groups);
        close(s);
        printf("\n");
}
```

To use it, we can slap it in an executable with a main like this (complete source [here](http://pintday.org/hack/code/ifgroupdemo.c "Source code for fun and profit")):


```C
int
main(int argc, char **argv)
{
        if (argc < 2)
                exit(0);
        grouplookup(argv[1]);
        exit(0);
}
```

And voila: the code works as expected:

    $ ifconfig ath0 group whatev
    $ ifconfig em0 group whatev
    $ ./a.out whatev
    ath0 em0

Ain't source code wonderful?
