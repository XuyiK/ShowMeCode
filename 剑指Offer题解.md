# 替换空格

```java
import org.junit.Test;

public class Solution {
    
	/**
     * 测试用例：
     * 1.输入的字符串包含空格（空格位于字符串的最前/最后/中间；字符串中有连续多个空格。
     * 2.输入的字符串没有空格。
     * 3.特殊输入测试：输入的字符串为null；输入的字符串是空字符串；字符串只有一个空格字符；字符串中有连续多个空格。
     */
    @Test
	public void testSolution(){
    	StringBuffer str1 = new StringBuffer(" Hello   World ");
    	StringBuffer str2 = new StringBuffer("HelloWorld");
    	StringBuffer str3 = null;
    	StringBuffer str4 = new StringBuffer("");
    	StringBuffer str5 = new StringBuffer(" ");
    	
		System.out.println(replaceSpace(str1));
		System.out.println(replaceSpace(str2));
		System.out.println(replaceSpace(str3));
		System.out.println(replaceSpace(str4));
		System.out.println(replaceSpace(str5));

	}
    
    
    public String replaceSpace(StringBuffer str) {
        // 边界检查!!!
    	if(str == null || str.length() <= 0) {
            return "";  // 根据情况选择错误返回值
        }
        int originalLength = str.length();
        int numberOfSpace = 0;
        // 记录一共有多少个空格
        for(int i = 0; i < originalLength; i++) {
            if(str.charAt(i) == ' ') {
                numberOfSpace++;
            }
        } 
        // 在数组后补足至足够放入整个数组
        str.setLength(originalLength + 2 * numberOfSpace);
        // 从尾部开始复制
        int indexOfOriginal = originalLength - 1;
        int indexOfNew = str.length()-1;
        while(indexOfOriginal >= 0 && indexOfNew > indexOfOriginal) {
        	if(str.charAt(indexOfOriginal) == ' ') {
                // 其实可以不用逐个替换，此处只是为了体现原著中的思想，
                // 实际上可以使用replace(int start, int end, String str)
        		str.setCharAt(indexOfNew--, '0');
        		str.setCharAt(indexOfNew--, '2');
        		str.setCharAt(indexOfNew--, '%');
        	} else {
        		str.setCharAt(indexOfNew, str.charAt(indexOfOriginal));
        		indexOfNew--;
        	}
            indexOfOriginal--;
        	       	
        }
        return str.toString();
    }
}
```

**:pencil2:举一反三**

在合并两个数组（包括字符串）时，如果从前往后复制每个数字（或字符）则需要移动数字（或字符）多次，那么我们可以考虑从后往前复制，这样就能减少移动的次数，从而提高效率。