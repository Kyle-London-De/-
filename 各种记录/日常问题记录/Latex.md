# 文字页边距正常，但图片超出页边距
原因：figure本身为浮动型，在`\begin{figure}`后加入`[bth]`等强制的位置参数导致图片超出页边距
解决方案：尝试移动图片位置，使之自然出现在希望出现的地方，如果还是不行，直接添加页边距设置
```latex
\usepackage[left=16.9mm,right=16.9mm,top=20.1mm,bottom=15.2mm]{geometry}
```