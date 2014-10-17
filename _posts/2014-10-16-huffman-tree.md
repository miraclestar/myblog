---
layout: default
title: 哈夫曼编码与ip库
category: tech
---
##{{ page.title }}
都懒了2个月了，这个月要出点东西了；

哈夫曼编码和ip库，这2个东西怎么还有关联？
本文就来说一下怎么用哈弗曼编码的方式存储ip库。

+ 首先要普及下ip知识，ip库一般是这样的格式：
>ip/mask isp
>##123.123.123.123/23 上海电信
>##12.23.123.1/24 北京网通

ip其实是32位二进制数表示的；
IP地址中间用点号隔开的4个十进制数，每个数取值范围是0—255，也就是256个数，是2的8次方，用2进制表示即为8位，整个ip即为32位二进制。
后面那个mask是子网掩码；表示一个子网；23表示23个1；ip和mask做“与”运算，得到的值相同那么2个ip段就是一个子网；
所以一个网段其实就是一堆2进制数；类似这样的100101010111；

+ ok，看出端倪了吗？别急，咱们再来普及下哈夫曼树的知识：
哈夫曼编码，简单来说，就是在一个哈夫曼二叉树左叶子标记0，右叶子标记1的或者左叶子标记1，右叶子标记0也行；
哈夫曼树就是一个二叉树，至于这个树怎么长出来的；

代码只写了怎么创建树，待日后再写从库里查询，其实就是顺藤摸瓜，就简单了；
代码先贴上，供大家参考，欢迎讨论:

    import java.util.HashSet;
    import java.util.Set;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    public class HuffmanIp {

    private static Logger log = LoggerFactory.getLogger(HuffmanIp.class);

    private HIpNode root = null;
    public static Set<HIpNode> leafNodes = new HashSet<HIpNode>();

    public HuffmanIp() {
        this.root = null;
    }

    public String getStrIpBy32Int(Long k) {
        String ip = "";
        Long p1 = ((k & 0xff000000) >> 24);
        Long p2 = ((k & 0x00ff0000) >> 16);
        Long p3 = (k & 0x0000ff00) >> 8;
        Long p4 = (k & 0x000000ff);
        ip = p1.toString() + "." + p2.toString() + "." + p3.toString() + "." + p4.toString();
        log.debug("ip string: " + ip);
        return ip;
    }

    public Long get32IntIpByIp(String ip) {
        Long ipInt = new Long(0);
        String[] ps = ip.split("\\.");
        if (4 != ps.length) {
            return ipInt;
        }
        Long p1 = new Long(ps[0]);
        Long p2 = new Long(ps[1]);
        Long p3 = new Long(ps[2]);
        Long p4 = new Long(ps[3]);
        ipInt = p1 * 256 * 256 * 256 + p2 * 256 * 256 + p3 * 256 + p4;
        log.debug("ipInt: " + ipInt);
        return ipInt;
    }

    private HIpNode insertNewNode(byte k, HIpNode n) {

        if (0 == k) {// left child

            if (null == n.left) {
                n.left = new HIpNode();
            }
            if (null != n.isp) { // split father to two leaf nodes
                n.right = new HIpNode();
                n.right.isp = n.isp;
                n.left.isp = n.isp;
                n.isp = null;
                leafNodes.remove(n);
                leafNodes.add(n.left);
                leafNodes.add(n.right);
            }
            return n.left;
        } else if (1 == k) {// right child

            if (null == n.right) {
                n.right = new HIpNode();
            }
            if (null != n.isp) {// split father to two leaf nodes
                n.left = new HIpNode();
                n.left.isp = n.isp;
                n.right.isp = n.isp;
                n.isp = null;
                leafNodes.remove(n);
                leafNodes.add(n.left);
                leafNodes.add(n.right);
            }
            return n.right;
        }

        return null;
    }

    public void insert(long ipLong, Integer mask, int ispid) {
        if (null == this.root) {

            this.root = new HIpNode();
            log.debug("new root: " + root);
        }

        HIpNode node = this.root;

        log.debug("Binary ip： " + Long.toBinaryString(ipLong) + ",mask: " + mask);

        ipLong = ipLong >> (32 - mask);
        log.debug(" ip >> 32-mask ,Binary subip： " + Long.toBinaryString(ipLong));

        for (int i = mask - 1; i >= 0; i--) {
            byte k = (byte) ((ipLong >> i) & 0x00000001);// 从高往低位，每次取一位ip；

            if (null != node.isp) {
                if (ispid == node.isp) {// same isp break;
                    break;
                }
            }
            node = insertNewNode(k, node);
        }

        node.isp = ispid;
    }

    public HIpNode getRoot() {
        return root;
    }

    public static void main(String args[]) {
        System.out.println(1110100010011l >> 13 & 0x00000001);
        HuffmanIp tmp = new HuffmanIp();
        Long x = tmp.get32IntIpByIp("116.76.102.0");
        Long y = tmp.get32IntIpByIp("116.76.0.0");
        int mask = 14;
        int ispid = 74;
        log.debug("long ip : " + x + ",mask:" + mask + ",ispid:" + ispid);
        tmp.insert(x, mask, ispid);
        tmp.insert(y, 15, 2);
        tmp.insert(y, 12, 90);
        tmp.insert(x, 15, 70);
        System.out.println(leafNodes);
    }
}
`


<br /><p>{{ page.date | date_to_string }}</p>
