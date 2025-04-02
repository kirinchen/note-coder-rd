---
title: 如何讓Gpt知道你的專案內容
tags: [AI, ChatGPT, OpenAI]

---

如何讓GPT了解你的專案
============

動機
--

在許多開發環境中，如Tabnine或GitHub Copilot提供代碼完成建議，但這些工具可能無法針對特定模型進行定制，或者需要付費使用。本教程將介紹如何通過自定義GPT模型，使其更深入地理解您的專案結構和代碼，從而提供更加精確的建議。

使用Shell腳本匯出專案所有檔案內容
-------------------

以下的shell腳本可以用來將專案目錄下的所有檔案內容匯出到一個文本文件中，排除一些常見的非源代碼目錄。

```bash=
#!/bin/bash

# Define the output file
output_file="all_files_contents.txt"

# Define excluded directories in a variable as an array
declare -a exclude_dirs=("bin" "build" "logs" "out" ".git" ".gradle" ".idea" "init-model" "resources")

# Build the find command exclude path argument dynamically
exclude_args=()
for dir in "${exclude_dirs[@]}"; do
    exclude_args+=(! -path "*/$dir/*")
done

# Clear the output file if it already exists
> "$output_file"

# Define a function to append file details to the output file
process_file() {
    local file_path="$1"
    echo "--------------" >> "$output_file"
    echo " ${file_path}" >> "$output_file"
    echo "--------------" >> "$output_file"
    cat "$file_path" >> "$output_file"

    echo "" >> "$output_file"
}

# Export the function so it can be used in a subshell
export -f process_file
export output_file

# Use find to locate all files, excluding specified directories and the output file itself
find . -type f "${exclude_args[@]}" ! -name "$(basename -- "$output_file")" -exec bash -c 'process_file "$0"' {} \;

echo "File processing complete. Output is in $output_file"

```

### 腳本解釋

-   輸出文件定義: 指定輸出文件名為 `all_files_contents.txt`。
-   排除目錄: 定義一個陣列來列出所有要排除的目錄，以避免將構建檔案或日誌等非源代碼包含在內。
-   動態構建排除參數: 使用陣列和迴圈構建 `find` 命令的排除參數。
-   清空輸出文件: 如果輸出文件已存在，則先清空。
-   處理文件的函數: 定義一個函數 `process_file`，將每個文件的路徑和內容追加到輸出文件中。
-   執行查找命令: 使用 `find` 命令查找所有文件，並調用 `process_file` 函數處理這些文件，同時排除已指定的目錄和輸出文件本身。

如何使用MyGPTs
----------

### 創建你的GPT模型

1.  前往 OpenAI 平台或任何支持自訂GPT模型的平台。
2.  上傳創建好的 `all_files_contents.txt` 文件到模型訓練平台。
3.  設定相關參數並開始訓練模型。

### 使用你的GPT模型

一旦模型訓練完成，你就可以在你的開發環境中調用這個模型來獲得代碼建議或進行其他相關的操作。

結論
--

通過這個教程，你可以學會如何讓一個自訂的GPT模型了解你的專案內容，從而在開發過程中提供針對性的支援。這不僅可以增強開發工具的智能，還能在某些程度上節省購買商業工具的成本。