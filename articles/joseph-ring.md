## 约瑟夫环

约瑟夫环（约瑟夫问题）是一个数学的应用问题：已知n个人（以编号1，2，3...n分别表示）围坐在一张圆桌周围。从编号为k的人开始报数，数到m的那个人出列；他的下一个人又从1开始报数，数到m的那个人又出列；依此规律重复下去，直到圆桌周围的人全部出列（或者求最后一个出列的人的编号）。

这里提供了三种方法：

逻辑去除法；

递归法；

线性表法。

这个问题我觉得应该也可以用循环队列来解决，这种方法以后再加上。

### 逻辑去除法

```
	/**
     * 逻辑去除法
     * @param $n
     * @param $m
     * @return int
     */
    public function ExJosephRing_Logic($n, $m)
    {
        $tmp_array = array();
        // $left_over记录数组中不为0的个数
        $left_over = $n;
        // $ret_value记录最后返回的数字
        $ret_value = 0;
        // 记录开始循环查找的起点
        $start = 0;
        // 初始化数组
        for ($i = 0; $i <= $n; $i++) {
            $tmp_array[$i] = $i;
        }
        $array_length = count($tmp_array);
        while ($left_over != 1) {
            for ($j = 1; $j <= $m; $j++) {
                $start++;
                // $tmp_array[$start] == 0检测数组越界
                if ($start != $array_length && $tmp_array[$start] == 0) {
                    for ($k = $start; $k < $array_length; $k++) {
                        if ($tmp_array[$k] == 0) {
                            $start++;
                        } elseif ($tmp_array[$k] > 0) {
                            break;
                        }
                    }
                }
                // 循环到头，重新开始
                if ($start == $array_length) {
                    $start = 0;
                }
                if ($tmp_array[$start] == 0) {
                    for ($k = $start; $k < $array_length; $k++) {
                        if ($tmp_array[$k] == 0) {
                            $start++;
                        } elseif ($tmp_array[$k] > 0) {
                            break;
                        }
                    }
                }
            }
            
            $tmp_array[$start] = 0;

            // 循环遍历，记录数组中不为0的元素的个数
            $left_over = 0;
            foreach ($tmp_array as $key => $value) {
                if ($value != 0) {
                    $left_over++;
                    $ret_value = $value;
                }
            }
        }
        return $ret_value;
    }
```

### 递归方法

```
	/**
     * 递归方法
     * @param $tmp_array
     * @param $m
     * @param int $current
     * @return int
     */
    public function ExJosephRing_Recursive($tmp_array, $m, $current = 0)
    {
        $array_length = count($tmp_array);
        $num = 1;
        if (count($tmp_array) == 1) {
            return $tmp_array[0];
        } else {
            while ($num++ < $m) {
                $current++;
                $current = $current % $array_length;
            }
            array_splice($tmp_array, $current, 1);
            // 如果需要返回数据，这里必须加上return，否则不会返回结果
            return $this->ExJosephRing_Recursive($tmp_array, $m, $current);
        }
    }
```

### 线性表法

```
	/**
     * 线性表法
     * @param $n
     * @param $m
     * @return int
     */
    public function ExJosephRing_Linear($n, $m)
    {
        $r = 0;
        for ($i = 2; $i <= $n; $i++) {
            $r = ($r + $m) % $i;
        }
        return $r + 1;
    }
```

对于线性表法的解释：每个人出列后，剩下的人又组成了另一个子问题。只是他们的编号变化了。第一个出列的人肯定是a[1] = m(mod)n（m/n的余数），他除去后剩下的人编号是a[1]+1、a[1]+2、…、n、1、2、…a[1]-2、a[1]-1，对应的新编号是1，2，3…n-1。设此时某个人的新编号是i，他原来的编号就是(i+a[1])%n。于是，这便形成了一个递归问题。假如知道了这个子问题（n-1个人）的解是x，那么原问题（n个人）的解便是：(x+m%n)%n = (x+m)%n。问题的起始条件：如果n=1,那么结果就是1。



Read More:

> [用PHP解决“约瑟夫环”的几种方法](http://9iphp.com/web/php/1112.html)