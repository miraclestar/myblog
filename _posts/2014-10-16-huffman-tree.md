---
layout: default
title: ������������ip��
category: tech
---
##{{ page.title }}

�����������ip�⣬��2��������ô���й�����
���ľ���˵һ����ô�ù���������ķ�ʽ�洢ip�⡣

+ ����Ҫ�ռ���ip֪ʶ��ip��һ���������ĸ�ʽ��
>ip/mask isp
>123.123.123.123/23 �Ϻ�����
>12.23.123.1/24 ������ͨ
ip��ʵ��32λ����������ʾ�ģ�
IP��ַ�м��õ�Ÿ�����4��ʮ��������ÿ����ȡֵ��Χ��0��255��Ҳ����256��������2��8�η�����2���Ʊ�ʾ��Ϊ8λ������ip��Ϊ32λ�����ơ�
�����Ǹ�mask���������룻��ʾһ��������23��ʾ23��1��ip��mask�����롱���㣬�õ���ֵ��ͬ��ô2��ip�ξ���һ��������
����һ��������ʵ����һ��2������������������100101010111��

+ ok�������������𣿱𼱣����������ռ��¹���������֪ʶ��
���������룬����˵��������һ����������������Ҷ�ӱ��0����Ҷ�ӱ��1�Ļ�����Ҷ�ӱ��1����Ҷ�ӱ��0Ҳ�У�
������������һ���������������������ô�������ģ�

����ֻд����ô�����������պ���д�ӿ����ѯ����ʵ����˳�����ϣ��ͼ��ˣ�
���������ϣ�����Ҳο�����ӭ����:
`
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

        log.debug("Binary ip�� " + Long.toBinaryString(ipLong) + ",mask: " + mask);

        ipLong = ipLong >> (32 - mask);
        log.debug(" ip >> 32-mask ,Binary subip�� " + Long.toBinaryString(ipLong));

        for (int i = mask - 1; i >= 0; i--) {
            byte k = (byte) ((ipLong >> i) & 0x00000001);// �Ӹ�����λ��ÿ��ȡһλip��

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
