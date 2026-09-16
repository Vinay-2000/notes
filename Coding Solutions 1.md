### Reverse a String (without built-in methods)

```
String a = "Vinay";  
StringBuilder reverse = new StringBuilder();  
for (int i = a.length() - 1; i >= 0; i--) reverse.append(a.charAt(i));  
System.out.println(reverse.toString());

//Other good solution
char[] arr = a.toCharArray();
int left = 0;
int right = arr.length - 1;
while (left < right) {
    char temp = arr[left];
    arr[left] = arr[right];
    arr[right] = temp;
    left++;
    right--;
}
System.out.println(new String(arr));

//Built in
1. new StringBuilder(str).reverse().toString();
2. str.chars().mapToObj(c -> String.valueOf((char) c)).reduce("", (a, b) -> b + a);
3. Collections.reverse(chars); String reversed = String.join("", chars);
```

### Check Palindrome

```
//For complex string with space and other characters 
String str = "A man, a plan, a canal: Panama"
int left = 0;
int right = str.length() - 1;

while (left < right) {
    while (left < right && !Character.isLetterOrDigit(str.charAt(left))) {
        left++;
    }

    while (left < right && !Character.isLetterOrDigit(str.charAt(right))) {
        right--;
    }

    if (Character.toLowerCase(str.charAt(left)) != Character.toLowerCase(str.charAt(right))) {
        return false;
    }

    left++;
    right--;
}
return true;

//Normal case
String x = "racecar";
int i = 0, j = x.length() - 1;
boolean flag = true;

while (i < j) {
    if (x.charAt(i) == x.charAt(j)) {
        i++;
        j--;
    } else {
        flag = false;
        break;
    }
}

System.out.println("Palindrome: " + flag);
-----------------------------------------
//Without using break

String x = "racecar";
int i = 0;
int j = x.length() - 1;

while (i < j) {
    if (x.charAt(i) != x.charAt(j)) {
        System.out.println("Palindrome: false");
        return; // will end method
    }
    i++;
    j--;
}
System.out.println("Palindrome: true");
----------------------------------------
//Why is this O(1) space? Didn't we create variables `i`, `j`, and `flag`? Those are just a constant number of primitive variables. Space complexity measures how extra memory grows with input size. Since the number of variables remains constant regardless of the string length, the extra space is O(1).

//Built-in
boolean isPalindrome = str.equals(new StringBuilder(str).reverse().toString()); //Has O(n) time and space complexity

```

### Count Vowels

```
//Best Solution
String str = "RACECAR";
int count = 0;
for (char c : str.toCharArray()) {
    char ch = Character.toLowerCase(c);  
	if (ch == 'a' ||  ch == 'e' ||  ch == 'i' ||  ch == 'o' ||  ch == 'u') {  
		count++;  
	}
System.out.println("Total Vowels Count: " + vowelsCount);

//Same using Switch
String str = "RACECAR";
int count = 0;
for (char c : str.toCharArray()) {
    switch (Character.toLowerCase(c)) {
        case 'a':
        case 'e':
        case 'i':
        case 'o':
        case 'u':
            count++;
    }
}

//Ones I came up initially - Hashet is extra space which can be avoided
Set<Character> set = Set.of('a', 'e', 'i', 'o', 'u');  
String x = "RACECAR";  
int vowelsCount = 0;  
for (char c : x.toCharArray()) {  
	c = Character.toLowerCase(c);  
	if (set.contains(c)) vowelsCount++;  
}  
System.out.println("Total Vowels Count: " + vowelsCount);  

//here toLowerCase creates a new String which can be avoided.
Set<Character> set = Set.of('a', 'e', 'i', 'o', 'u');  
String x = "RACECAR";  
int vowelsCount = 0;  
for (char c : x.toLowerCase().toCharArray()) {  
    if (set.contains(c)) vowelsCount++;  
}  
System.out.println("Total Vowels Count: " + vowelsCount);
```

### Find Duplicate Characters

```
Set<Character> set = new HashSet<>();  
String x = "RACECAR";  
for (char c : x.toCharArray()) {  
    if (set.contains(c)) System.out.println("Duplicate Found: " + c);  
    else set.add(c);  
}  
//To ignore case we can do Character.toLowerCase() question doesn't specify this so not using this

//If you know the input is ASCII - ASCII will have exact 256 characters
boolean[] seen = new boolean[256];

for (char c : str.toCharArray()) {
    if (seen[c]) {
        System.out.println("Duplicate: " + c);
    } else {
        seen[c] = true;
    }
}
```


### Remove Duplicate Characters  

```
Set<Character> set = new HashSet<>();  
String x = "RACECAR";  
StringBuilder result = new StringBuilder();  
for (char c : x.toCharArray()) {  
    if (!set.contains(c)) {  
        set.add(c);  
        result.append(c);  
    }  
}  
System.out.println("No Duplicate: " + result.toString());

//Slight Optimized if condition
if (set.add(c)) {  
result.append(c);  
}
//the add returns if it got added or not and if its not duplicate will add return true 
//we can use that to append to string this way we save one set lookup

//Stream solution
String result = x.chars()
        .distinct()
        .mapToObj(c -> String.valueOf((char) c))
        .collect(Collectors.joining());
```

### First Non-Repeating Character

```
//Here expectation that character is only once in entire string and if multiple are there print first one
//First unique character
String x = "RRRACECAR";
//Count frequency
Map<Character, Integer> freq = new HashMap<>();
for (char c : x.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}
//Find the first character with frequency 1
for (char c : x.toCharArray()) {
    if (freq.get(c) == 1) {
        System.out.println(c);
        break;
    }
}
```

### Character Frequency Map
```
//Character Frequency Map  
String str = "abcadsadb";  
Map<Character, Integer> map1 = new HashMap<>();  
for (char i : str.toCharArray()) {  
    map1.put(i, map1.getOrDefault(i, 0) + 1);  
}  
System.out.println(map1);  
  
//Using streams  
Map<Character, Integer> map2 = str.chars().mapToObj(c -> (char) c)  
        .collect(Collectors.toMap((c) -> c, (c) -> 1, (old, newVal) -> old + newVal));  
System.out.println(map2);  

//Using groupingBy  
Map<Character, Long> map3 = str.chars().mapToObj(c -> (char) c).collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));  
System.out.println(map3);
```

### Check Anagram

```
//Most Optimized
int[] freq = new int[26];

for (char c : s.toCharArray()) {
    freq[c - 'a']++;
}

for (char c : t.toCharArray()) {
    freq[c - 'a']--;
}

for (int count : freq) {
    if (count != 0) {
        return false;
    }
}

return true;
```
```
 public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }
        // Map<Character, Long> map1 = s.chars().mapToObj((c) -> (char) c)
        //         .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
        // Map<Character, Long> map2 = t.chars().mapToObj((c) -> (char) c)
        //         .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));

        Map<Character, Integer> map1 = new HashMap<>();
        Map<Character, Integer> map2 = new HashMap<>();

        for (char c : s.toCharArray()) {
            map1.put(c, map1.getOrDefault(c, 0) + 1);
        }

        for (char c : t.toCharArray()) {
            map2.put(c, map2.getOrDefault(c, 0) + 1);
        }

        return map1.equals(map2);
    }
```

### Longest Common Prefix

```
//Best - Shrink till prefix match for each string in array
 public String longestCommonPrefix(String[] strs) {
        String res = strs[0];
        for (String s : strs) {
            while (!s.startsWith(res)) {
                if (res.length() == 0) return "";
                res = res.substring(0, res.length() - 1);
            }
        }
        return res;
    }
    
//Mine
  public String longestCommonPrefix(String[] strs) {
        String res = strs[0];

        for(int i=1; i< strs.length; i++){
            if(res.isEmpty()) break;
            int len = Math.min(strs[i].length(),res.length());
            int j =0;
            while(j<len && res.charAt(j) == strs[i].charAt(j)){
                j++;
            }
            res = res.substring(0,j);
        }
        return res;
    }   
    
```

### String Compression
```
class Solution {

    public int compress(char[] chars) {
        char curr = chars[0];
        int count = 1;

        int left = 0;
        int right = 1;
        while (right < chars.length) {
            if (chars[right] == curr) {
                count++;
                right++;
            } 
            else{
                chars[left] = curr;
                left++;
                if (count > 1) {
                    for (char a : String.valueOf(count).toCharArray()) {
                        chars[left] = a;
                        left++;
                    }
                }
                curr = chars[right];
                count = 0;
            }
            
        }
        chars[left] = curr;
                left++;
                if (count > 1) {
                    for (char a : String.valueOf(count).toCharArray()) {
                        chars[left] = a;
                        left++;
                    }
                }
        return left;
        
    }
}
```

### Find Second Largest

```
class Solution {
    public int getSecondLargest(int[] arr) {
        // code here
        int max = -1;
        int secondMax=-1;
        
        for(int i=0; i<arr.length; i++){
            if(arr[i] > max){
                secondMax = max;
                max = arr[i];
            }
            else if(arr[i] > secondMax && arr[i] !=max ){
                secondMax = arr[i];
            }
        }
        return secondMax;
    }
}
```

### Remove Duplicates in sorted array

```
class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length == 1) return 1;

        int left = 0;
        int right = 0;
        while(right < nums.length){
            if(nums[right] == nums[left]){
                right++;
            }
            else{
                left++;
                if(left!=right){
                    nums[left] = nums[right];
                }
                right++;
            }
        }
        return left+1;
    }
}
```


### Rotate Array

```
class Solution {
    public void rotate(int[] nums, int k) {
        int n = nums.length;
        k = k % n;

        reverse(nums, 0, n - 1);
        reverse(nums, 0, k - 1);
        reverse(nums, k, n - 1);
    }

    private void reverse(int[] nums, int left, int right) {
        while (left < right) {
            int temp = nums[left];
            nums[left] = nums[right];
            nums[right] = temp;

            left++;
            right--;
        }
    }
}

Other solutions
public void rotate(int[] nums, int k) {
        if(nums.length == 1) return;
         k = k % nums.length;
        int temp1 = nums[0];
        int temp2;
        
        for (int j = 0; j < k; j++) {
            for (int i = 1; i < nums.length; i++) {
                temp2 = nums[i];
                nums[i] = temp1;
                temp1 = temp2;
            }
            nums[0] = temp1;
        }
        //Using extra array
        int[] res = new int[nums.length];
        for (int i = 0; i < nums.length; i++) {
            if(i+k >= nums.length){
                int index = (i+k) % nums.length;
                res[index]  = nums[i];
            }
            else res[i+k] = nums[i];

        }
         for (int i = 0; i < nums.length; i++) {
             nums[i] = res[i];
         }
    }
```
### Move Zeros to End

```
 public void moveZeroes(int[] nums) {
        int left = 0;
        int right =0;

        while(right < nums.length){
            if(nums[right] != 0){
                int temp = nums[left];
                nums[left] = nums[right];
                nums[right] = temp;
                left++;
                right++;
            }else{
                right++;
            }
        }
    }
```

### Merge Two Sorted Arrays
```
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1;          // last valid element in nums1
        int j = n - 1;          // last element in nums2
        int k = m + n - 1;      // position to fill

        while (i >= 0 && j >= 0) {
            if (nums1[i] > nums2[j]) {
                nums1[k] = nums1[i];
                i--;
            } else {
                nums1[k] = nums2[j];
                j--;
            }
            k--;
        }

        while (j >= 0) {
            nums1[k] = nums2[j];
            j--;
            k--;
        }
    }
}
```

### Maximum Subarray
```
 public int maxSubArray(int[] nums) {
       int max = Integer.MIN_VALUE;
        int curr = 0;
       for(int i=0; i < nums.length; i++){
            curr += nums[i];
            max = Math.max(max, curr);
            if(curr < 0) curr = 0;
       } 
       return max;
    }
```

### Stock Buy & Sell
```
public int maxProfit(int[] prices) {
        int max = 0;
        int low = prices[0];
        for(int i=0; i < prices.length; i++){
            if(prices[i] < low) low = prices[i];
            else max = Math.max(prices[i] - low, max);
        }
        return max;
    }
```

### Majority Element

```
public int majorityElement(int[] nums) {
        int count = 1;
        int curr = nums[0];
        for(int i=1; i < nums.length; i++){
            if(nums[i] != curr) count--;
            else count++;
            
            if (count == 0){
                count =1;
                curr = nums[i];
            }
        }
        return curr;
    }
```