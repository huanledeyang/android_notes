# ConstraintLayout 
## 优点
- 嵌套少
- UI tools
- 自动化
> **引用**

![图片](http://mmbiz.qpic.cn/mmbiz_gif/v1LbPPWiaSt6amUiacxJ04yNrplc6AkQ5sS7yvNWHViauZbZpS86ejFPtGE0GVrV10BDgRTpB35hWfdE5gPDr54dw/0?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

### 代码
***
'	void quick_sort(int[] data, int start, int end)
{
	if(start >= end)
		return;
	
	int i = start;
	int j = end;
	int key = data[start];

	while(1)
	{
		while(data[i] < key) i++;
		while(data[j] > key ) j--;

		if(i >= j) break;
		
		int temp = data[i];
		data[i] = data[j];
		data[j] = temp;
		
		i++;
		j--;
	}
	
	quick_sort(data,start,i-1);
	quick_sort(data,j+1,end);
}'
