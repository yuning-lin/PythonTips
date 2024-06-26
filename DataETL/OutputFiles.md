回顧 pandas 如何[讀檔](https://github.com/yuning-lin/PythonTips/blob/main/DataETL/ReadFiles.md)  
pandas 大部分的檔案格式都可以儲存  
以下羅列使用過的讀檔方式以及曾經遇過的問題做整理：  
```python
import pandas as pd
```



## .csv
是最常使用的格式之一
```python
## 一般存檔方式
final_df.to_csv('output.csv', index=False)

## 有中文在檔案中
final_df.to_csv('output.csv', index=False, encoding="utf-8")
final_df.to_csv('output.csv', index=False, encoding="utf_8_sig")
```


## .xlsx
* [Doc：xlsxwriter](https://xlsxwriter.readthedocs.io/)
* 儲存 df_dict 中的 key 做為 sheet name、value 為 dataframe 存於多個 sheet
    ```python
    with pd.ExcelWriter(path) as writer:
        for key_name, dframe in df_dict.items():
            dframe.to_excel(writer, sheet_name=key_name, index=False)
    ```
* 客製化欄位設置：[add_format](https://xlsxwriter.readthedocs.io/format.html)
    1. 凍結欄位
    2. 數值欄位帶千分位逗號
    3. 調整欄寬
    ```python
    col_lst = df.columns.tolist()
    writer = pd.ExcelWriter(file_name, engine='xlsxwriter')
    df.to_excel(writer, sheet_name=sheet_name, freeze_panes=(1, 9), index=False) # freeze_panes=(1, 9)：凍結欄位名稱及左邊九欄
    workbook = writer.book
    worksheet = writer.sheets[sheet_name]

    num_format = workbook.add_format({'num_format': '#,##0.00'}) # 將數值欄位轉成千分位帶逗號，並取到小數點第二位
    for idx in range(len(col_lst)):
        worksheet.set_column(idx, idx, 10, num_format) # 將欄寬設為：10

    writer.save()
    ```
    錯誤訊息
    ```python
    AttributeError: ‘Worksheet’ object has no attribute 'set_column'
    ```
    解方
    ```python
    pip install XlsxWriter
    ```
* 鎖住欄位（有 cell > row > col 這樣可以 overwrite 的關係）
    1. 鎖欄
    2. 鎖列
    3. [overwrite 的關係](https://stackoverflow.com/questions/56240667/not-able-to-unlock-cell-with-custom-value-using-pd-xlsxwriter)
    4. [protect 的調整選項](https://xlsxwriter.readthedocs.io/worksheet.html)
    ```python
    writer = pd.ExcelWriter(file_name, engine='xlsxwriter')
    df.to_excel(writer, sheet_name='Sheet1', index=False)

    workbook  = writer.book
    worksheet = writer.sheets['Sheet1']

    unlocked = workbook.add_format({'locked': False})
    worksheet.protect()
    worksheet.set_column('A:E', None, unlocked) # 鎖欄
    for row in range(1, 150):
        worksheet.set_row(row, None, unlocked) # 鎖列

    writer.save()
    ```
* 隱藏欄位
    ```python
    worksheet.set_column("A:D", None, None, {'hidden': True})

    writer.save()
    ```
* 隱藏 sheet（建議要隱藏的 sheet 放後頭做）
    ```python
    writer = pd.ExcelWriter(file_name, engine='xlsxwriter')
    df.to_excel(writer, sheet_name='Sheet1', index=False)
    worksheet = writer.sheets['Sheet1']

    df.to_excel(writer, sheet_name='Sheet2', index=False)
    worksheet = writer.sheets['Sheet2']
    worksheet.hide()

    writer.save()
    ```
* [Example: Data Validation and Drop Down Lists](https://xlsxwriter.readthedocs.io/example_data_validate.html)
* [Example: DATA VALIDATION IN EXCEL](https://cxn03651.github.io/write_xlsx/data_validation.html)
    ```python
    import xlsxwriter
    # validate 選項：any, integer, decimal, list, date, time, length, custom
    col_content_dict = {
        "Name": {"validate": "list", "source": ["Alex", "Amy", "Alen"]},
        "City": {"validate": "list", "source": ["Taipei", "Tainan", "Taichung"]},
        "Height": {"validate": "integer", "source": None}
    }

    for col_idx, col_name in enumerate(df.columns.values):
        worksheet.write(0, col_idx, col_name, None)
        col_letter = xlsxwriter.utility.xl_col_to_name(col_idx) # idx 轉字母
        worksheet.data_validation(f'{col_letter}2:{col_letter}200', col_content_dict[col_name])
        worksheet.set_column(col_idx, col_idx, 10)
    ```
    錯誤訊息
    ```python
    UserWarning: Length of list items exceeds Excel's limit of 255, use a formula range instead
    ```
    解方
    ```python
    n = 100 # 值會寫入的欄
    city_lst = ["Taipei", "Tainan", "Taichung"]
    for row_count in range(0, len(city_lst)):
        worksheet.write(row_count, n, city_lst[row_count]) # 將所有值寫入該欄
    
    worksheet.data_validation(f'B2:B200', {"validate": "list", "source": '=CWCW1:CWCW3'}) # 利用引用該欄所有值的方式創造下拉選單
    worksheet.data_validation(f'B2:B200', {"validate": "list", "source": '=sheet1!AA1:AA3'}) # 利用引用 sheet1 A欄所有值的方式創造下拉選單
    ```
* 顯示篩選符號
    ```python
    writer = pd.ExcelWriter('testtest.xlsx', engine='xlsxwriter')
    df.to_excel(writer, sheet_name='Sheet1')
    worksheet = writer.sheets['Sheet1']
    worksheet.autofilter('A1:BX1')
    ```
* 改指定位置背景顏色
    ```python
    writer = pd.ExcelWriter('testtest.xlsx', engine='xlsxwriter')
    workbook = writer.book
    worksheet = workbook.add_worksheet()
    # 用矩陣位置表示來看，從 [0,2] 到 [1,4] 的位置統一填上背景色
    worksheet.conditional_format(0, 2, 1, 4, {'type': 'no_errors', 'format': workbook.add_format({'bg_color': 'red'})})
    ```  
* 改指定位置文字顏色
    ```python
    from openpyxl import Workbook  
    from openpyxl.styles import Font, Color  
      
    wb = Workbook()  
    ws = wb.active  
      
    # 創建一個有兩種顏色的字  
    font = Font(color="000000")  # 黑色  
    red_font = Font(color="FF0000")  # 紅色  
    ws['A1'].value = "所有字都黑的但最後一個字變紅色"  
      
    # 把最後一個字變紅色  
    ws['A1'].font = font  
    ws['A1'].value[-1].font = red_font  
      
    wb.save('sample.xlsx')
    ```
    ```python
    writer = pd.ExcelWriter('sample.xlsx', engine='xlsxwriter')
    # 定義 excel format
    workbook = writer.book
    workbook.formats[0].set_font_name('Arial')  # 設全域字體
    worksheet = workbook.add_worksheet()

    # 定義格式
    format_black = workbook.add_format({'color': 'black', 'align': 'center'})
    format_red = workbook.add_format({'color': 'red', 'align': 'center'})

    # 指定某個單元格的文字，Hello 為黑色、World 為紅色
    worksheet.write_rich_string(f'A1', format_black, 'Hello', format_red, 'World')
    ```
* 合併儲存格並置中文字
    ```python
    from openpyxl import Workbook  
    from openpyxl.styles import Alignment  
    # 建立一個新的工作簿  
    wb = Workbook()  
    # 選擇工作簿的活動工作表  
    ws = wb.active  
      
    # 為 A1 儲存格添加文字  
    ws['A1'] = 'Hello World'  
    # 合併 A1 和 B1  
    ws.merge_cells('A1:B1')  
    # 設定 A1 儲存格的對齊方式為置中  
    ws['A1'].alignment = Alignment(horizontal='center')  
      
    # 儲存檔案  
    wb.save('sample.xlsx')  
    ```

    ```python  
    import xlsxwriter  
    
    # 創建一个新的Excel文件並添加一工作表。  
    workbook = xlsxwriter.Workbook('sample.xlsx')  
    worksheet = workbook.add_worksheet()  
    
    # 新增格式：藍色背景色、文字置中。  
    format = workbook.add_format({'bg_color': '#00B0F0', 'align': 'center'})  
    
    # 合併指定儲存格，並應用上述格式。  
    worksheet.merge_range('B3:D3', 'Merged Range', format)  
    
    workbook.close()  
    ```

* 列顏色交錯
    * 若僅橫列需要控制格式，則可使用 set_row
    ```python
    import xlsxwriter
    from itertools import cycle
    
    writer = pd.ExcelWriter('testtest.xlsx', engine='xlsxwriter')
    df.to_excel(writer, sheet_name='Sheet1')
    worksheet = writer.sheets['Sheet1']
    # 法一：cycle
    data_format1 = workbook.add_format({'bg_color': '#EEEEEE'})
    data_format2 = workbook.add_format({'bg_color': '#DDDDDD'})
    data_format3 = workbook.add_format({'bg_color': '#CCCCCC'})
    formats = cycle([data_format1, data_format2, data_format3])
    for row, value in enumerate(data):
        data_format = next(formats)
        worksheet.set_row(row, cell_format=data_format)
    
    # 法二：邏輯判斷
    bg_format1 = workbook.add_format({'bg_color': '#78B0DE'}) # blue cell background color
    bg_format2 = workbook.add_format({'bg_color': '#FFFFFF'}) # white cell background color
    for i in range(10): # integer odd-even alternation 
        worksheet.set_row(i, cell_format=(bg_format1 if i%2==0 else bg_format2))
    ```
    * 若直行橫列皆有格式設定，會先後彼此覆蓋格式，則建議用：add_table（可將滑鼠移置 excel 模板 > 設計 > 表格樣式 > 停留幾秒，小白框會顯示 style）
    ```python
    from xlsxwriter.utility import xl_range
    df.T.reset_index().T.to_excel(
        writer, sheet_name='Sheet1', index=False, header=None)
    worksheet = writer.sheets['Sheet1']

    end_row = len(df.index)
    end_column = len(df.columns) - 1
    cell_range = xl_range(0, 0, end_row, end_column) # 指定表格位置
    worksheet.add_table(cell_range, {'data': df.values.tolist(),
                                     'columns': [{'header': c} for c in df.columns.tolist()],
                                     'style': 'Table Style Medium 2'})
    ```
