## 使用套件：`re`

## 進階用法
* 擷取文章中的句子
    * 斷句包含標點符號，並根據中英文選取不同標點符號
    * 使用 re.compile、findall
    * **注：中文標號在 linux 系統跑可能會有問題（待解）**
    ```python
    ## 英文
    txt = 'i am smart. you are fool! Are you sure?'
    pattern = re.compile('[\w\d\s]+[;,!\?\.]')
    sentence_lst = pattern.findall(txt)
    ```
    ```python
    ## 中文
    txt = '我很聰明，你是蠢蛋！你確定嗎？（我不確定）。'
    pattern = re.compile('[\w\d\s\（\）]+[；，！？。]')
    pattern.findall(txt)
    ```
    * 斷句不包含標點符號
    * 使用 re.split
    ```python
    txt = '我很聰明，你是蠢蛋！你確定嗎？（我不確定）。'
    re.split(r'(;|,|!|\?|\.|；|，|！|？|。)\s*', txt) # ()保留標點符號
    re.split(r'[;|,|!|\?|\.|；|，|！|？|。]\s*', txt) # []去除標點符號
    ```

* 擷取文字中的 email
    * 將完整 email 找出：`tutor@python.org`
    ```python
    txt = "To contact us, try tutor@python.org or the previous address tutor@google.com."
    pattern = re.compile(r'[\w.]+@\w+\.[a-z]{3}')
    pattern.findall(txt)
    ```
    * 利用小括號將搜索字詞分組輸出：`(tutor, python, org)`
    ```python
    txt = "To contact us, try tutor@python.org or the previous address tutor@google.com."
    pattern = re.compile(r'([\w.]+)@(\w+)\.([a-z]{3})')
    pattern.findall(txt)
    ```

* 擷取括號中的文字
   * 最小匹配（符合 pattern 下最短的字符）：`['小明', '阿明']`
   * 貪婪匹配（符合 pattern 下最長的字符）：`['小明)(阿明']`
    ```python
    """
    * r 可以避免使用反斜線表達特殊符號
    * [] 包住左右括弧表示此處的括弧為字符不是功能
    * . 是換行符號外的任一字元；* 表示出現 0 次～無限次，即括號中間有沒有文字都可以；? 表最小匹配
    * re.S 讓 . 表示是換行符號外的任一字元
    """
    txt = "王小明(小明)(阿明)"
    p1 = re.compile(r'[(](.*?)[)]', re.S) # 最小匹配
    p2 = re.compile(r'[(](.*)[)]', re.S) # 貪婪匹配
    re.findall(p1, txt)
    re.findall(p2, txt)
    ```
