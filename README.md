## 先预览一下成果吧~


<br/>
<br/>
 
> ## 1. 批量删除代码中的注释 
> ## 2. 支持多种文件后缀 （.java、.cs、.aspx、.html、.vue、.js、.css、.wxml、...）
> ## 3. 支持多种注释类型（//、///、/* ... */  、< !-- ... -->  、<%-- ... --%> ...）单行多行块注释 
> ## 4. PS:容易误伤某些代码(比如自动生成的、网址包含//)  
> ## ***请使用前备份！请选择自己代码目录！自动生成的代码不要使用***

<br/>
<br/>

实现思路: 
循环目录下指定后缀的子文件 .cs .vue .js.....  
一行一行读取内容 检测到指定注释类型 // /* ... 就去除注释内容
最后重写入文件


附上实现代码（C#控制台）：

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace DeleteNotes
{
    class Program
    {
        static void Main(string[] args)
        {
            Delete Delete = new Delete();

            Console.WriteLine("----------欢迎使用诗肴小工具----------");

            while (true)
            {
                Console.WriteLine("");
                Console.WriteLine("-----------请输入文件夹路径-----------");
                String Path = Console.ReadLine();

                //循环项目文件并删除注释
                Delete.DeleteNotes(Path);

                Console.WriteLine("");
                Console.WriteLine("--------------操作文件结束，结果统计如下：--------------");
                Console.WriteLine("");
                Console.WriteLine("项目文件数：" + Delete.FileAmount + " ，指定后缀文件：" + Delete.SuccessAmount + " ，操作成功：" + Delete.SuccessAmount + " ，操作失败：" + Delete.FailAmount);


                //文件数量
                Delete.FileAmount = 0;
                //操作文件数量
                Delete.HasActionFileAmount = 0;
                //成功文件数量
                Delete.SuccessAmount = 0;
                //失败文件数量
                Delete.FailAmount = 0;
            }

        }

    }

    /// <summary>
    /// 删除类
    /// </summary>
    public class Delete
    {
        /// <summary>
        /// 指定文件后缀 带点扩展名 ".xxxx" 列表
        /// </summary>
        List<String> ExtensionList = new List<string>();

        //文件数量
        public Int32 FileAmount = 0;
        //操作文件数量
        public Int32 HasActionFileAmount = 0;
        //成功文件数量
        public Int32 SuccessAmount = 0;
        //失败文件数量
        public Int32 FailAmount = 0;

        /// <summary>
        /// 构造方法
        /// </summary>
        public Delete()
        {
            //文件数量
            FileAmount = 0;
            //操作文件数量
            HasActionFileAmount = 0;
            //成功文件数量
            SuccessAmount = 0;
            //失败文件数量
            FailAmount = 0;

            #region 指定文件后缀
            //添加需要删除注释的文件后缀
            ExtensionList.Add(".vue");
            ExtensionList.Add(".js");
            ExtensionList.Add(".css");
            ExtensionList.Add(".scss");
            ExtensionList.Add(".wxss");
            ExtensionList.Add(".wxml");
            //ExtensionList.Add(".json"); json类型暂不处理
            ExtensionList.Add(".cs");
            ExtensionList.Add(".xml");
            ExtensionList.Add(".html");
            ExtensionList.Add(".asax");
            ExtensionList.Add(".Master");
            ExtensionList.Add(".aspx");
            ExtensionList.Add(".java");
            #endregion

        }

        /// <summary>
        /// 循环项目文件并删除注释
        /// 实现原理:
        /// 循环项目所有指定后缀的子文件 .cs .vue .js.....
        /// 然后一行一行读取内容 如果检测到 // /* ... 就剔除
        /// 最后重写文件
        /// </summary>
        public void DeleteNotes(String FilePath)
        {
            try
            {

                #region 循环项目文件
                //实例化项目目录
                DirectoryInfo MyDirectory = new DirectoryInfo(FilePath);
                //获取目录下所有文件和子目录
                FileSystemInfo[] FileList = MyDirectory.GetFileSystemInfos();
                //循环文件
                foreach (FileSystemInfo Item in FileList)
                {
                    FileSystemInfo FileInfo = Item;
                    //判断是否为文件夹
                    if (FileInfo is DirectoryInfo)
                    {
                        //递归调用
                        DeleteNotes(FileInfo.FullName);
                    }
                    else
                    {
                        //文件数量
                        FileAmount++;
                        //获取文件后缀 带点扩展名 ".xxxx"
                        String Extension = Path.GetExtension(FileInfo.FullName);
                        //指定后缀类型文件才操作
                        if (ExtensionList.Contains(Extension))
                        {
                            //操作文件数量
                            HasActionFileAmount++;
                            //--------开始处理文件-----------

                            try
                            {
                                #region 操作文件
                                //操作单行内容
                                //读取文件内容 一次读取一行
                                StreamReader StreamReader = new StreamReader(FileInfo.FullName);
                                StringBuilder NewContent = new StringBuilder();
                                //内容行
                                String Line;
                                //是否进入块注释
                                Boolean HasLumpNote = false;
                                while ((Line = StreamReader.ReadLine()) != null)
                                {
                                    //是否进入块注释
                                    if (HasLumpNote)
                                    {
                                        //1.进入块注释 检查块注释是否结束
                                        if (Line.IndexOf("*/") >= 0 || Line.IndexOf("-->") >= 0 || Line.IndexOf("--%>") >= 0)
                                        {
                                            //块注释结束
                                            HasLumpNote = false;
                                        }
                                    }
                                    else
                                    {
                                        //2.未进入块注释
                                        #region 判断是否包含块注释 比如: /* ... */  、<!-- ... -->  、<%-- ... --%> 
                                        //判断是否包含块注释 
                                        if (Line.IndexOf("/*") >= 0 || Line.IndexOf("<!--") >= 0 || Line.IndexOf("<%--") >= 0)
                                        {
                                            //进入块注释
                                            HasLumpNote = true;

                                            //判断块注释是否结束  这里有可能块注释就一行 开始和结束都在一行
                                            if (Line.IndexOf("*/") >= 0 || Line.IndexOf("-->") >= 0 || Line.IndexOf("--%>") >= 0)
                                            {
                                                //未进入块注释
                                                HasLumpNote = false;

                                                //注释出现位置下标
                                                Int32 Index =
                                                    Line.IndexOf("/*") < 0 ?
                                                        Line.IndexOf("<!--") < 0 ?
                                                            Line.IndexOf("<%--") < 0 ?
                                                            0
                                                            : Line.IndexOf("<%--")
                                                        : Line.IndexOf("<!--")
                                                    : Line.IndexOf("/*");

                                                //追加代码内容 直到注释出现位置
                                                NewContent.AppendLine(Line.Substring(0, Index < 0 ? 0 : Index));
                                            }
                                        }
                                        else
                                        {
                                            //3.不包含块注释 再判断一下行注释( 比如://、///)
                                            if (Line.IndexOf("//") < 0)
                                            {
                                                //未进入块注释 也无行注释 则追加代码内容 
                                                NewContent.AppendLine(Line);
                                            }
                                            else
                                            {
                                                //未进入块注释 有行注释 则追加代码内容直到注释出现位置
                                                NewContent.AppendLine(Line.Substring(0, Line.IndexOf("//")));
                                            }

                                        }
                                        #endregion
                                    }
                                }

                                //释放资源
                                StreamReader.Close();
                                //重新写入文件内容
                                File.WriteAllText(FileInfo.FullName, NewContent.ToString());
                                //成功文件数量
                                SuccessAmount++;
                                //Console.WriteLine("操作文件: 《" + FileInfo.FullName + "》 成功");

                                #endregion

                            }
                            catch (Exception err)
                            {
                                //失败文件数量
                                FailAmount++;
                                Console.WriteLine("操作文件: 《" + FileInfo.FullName + "》 失败,原因:" + err.Message);
                            }

                            //--------结束处理文件-----------

                        }
                    }
                }
                #endregion

            }
            catch (Exception err)
            {
                Console.WriteLine("操作失败,原因:" + err.Message);
            }
        }

    }
}


```

