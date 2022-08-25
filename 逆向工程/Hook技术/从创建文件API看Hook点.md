## 1、缘起

在IAT实验中， 我们通过Hook了kernel32.dll中的API：CreateFileW来修改创建文件的路径和文件名。从CreateFile整个生命周期来看，我们选择的Hook只是在其最表面的一层。为了更加深入了解除了Hook CreateFile这个API之外，还能通过Hook哪些API达到同样的效果。

- createfile生命周期
- CreateFile Hook点

## 2、CreateFile生命周期

### 2.1 第一层

从一个简单的用户层应用开始。此代码功能实现文件创建。API：CreateFile

```C++
void CTaregetDlg::OnBnClickedCreate()
{
	// TODO: 在此添加控件通知处理程序代码
	CString  filename;
	GetDlgItem(IDC_FILENAME)->GetWindowTextW(filename);
	CString  targetDir = L"E:\\IATHook\\" + filename;
	HANDLE hfile = NULL;
	if (filename.GetLength() != 0) {
		//MessageBox(filename, L"测试", MB_OKCANCEL);
		hfile = CreateFileW(targetDir, GENERIC_READ, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, 0);
		if (hfile == INVALID_HANDLE_VALUE)
		{
			MessageBox(L"创建文件失败", L"测试", MB_OKCANCEL);
		}
	}
	CloseHandle(hfile);

}
```

应用层有两个不同版本的CreateFile，其实质时一个API

```C++
#ifdef UNICODE
#define CreateFile  CreateFileW
#else
#define CreateFile  CreateFileA
#endif // !UNICODE
```



### 2.2 第二层