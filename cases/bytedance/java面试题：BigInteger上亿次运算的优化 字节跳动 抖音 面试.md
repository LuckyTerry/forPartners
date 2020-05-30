# java面试题：BigInteger上亿次运算的优化 字节跳动 抖音 面试

https://blog.csdn.net/m0_37877487/article/details/83859763

待改进
使用到BigInteger时，肯定要进行大量的运算，但面试考试要求时间在一秒以内；

经检验，一亿的百万次除法运算和百万的百万次除法运算耗时差距不大，但一亿的亿次运算和百万次运算差距很大。

所以大数运算不需要解决，要解决的是如何减少运算次数。

例题是要计算a/1 +a/2+·····+a/b求和

可以知道当100亿除以50亿以上的数字结果都是一，100亿除以3333···3到50亿的数字结果都是2；

可以先用上一行的规则进行10万次运算把要求的数字从100亿萎缩到10万，然后在进行暴力运算十万次

原本要运行100亿次，现在只运算了10万+N*10万次

经过实际验证，百亿次计算只需要100ms，如果完全硬算，只要一亿次运算就需要8000+ms。

所以最粗糙的一个算法改进使得百亿次运算速度提高了8000倍左右。

面试题 BigInteger a/1 +a/2+·····+a/b  求和  java

/**
 * biginteger 不要节约命名
 */
package 考试字节跳动;
 
import java.math.BigInteger;
import java.util.Scanner;
 
public class Z05 {
	public static void main(String[] args) {
		@SuppressWarnings("resource")
		Scanner scanner = new Scanner(System.in);
		int youhua = scanner.nextInt();
		long time1 = System.currentTimeMillis();
 
		BigInteger a = new BigInteger("5");
		BigInteger b = new BigInteger("10000000000");
		BigInteger after = b;
		BigInteger yi = new BigInteger("1");
		BigInteger count = new BigInteger("0");
		BigInteger before = new BigInteger("0");
		
		if (b.compareTo(a)>0) {
			after = a;
		}
 
		// 优化
		BigInteger flag = new BigInteger("2");
		for (int i = 0; i < youhua; i++, flag = flag.add(yi)) {
			before = after;
			after = a.divide(flag);
			count = count.add(flag.subtract(yi).multiply(before.subtract(after)));
		}
 
		do {
			count = count.add(a.divide(after));
			after = after.subtract(yi);
		} while (!(after.compareTo(yi) == -1));
 
		long time = System.currentTimeMillis() - time1;
		System.out.println(time + " " + count);
	}
}