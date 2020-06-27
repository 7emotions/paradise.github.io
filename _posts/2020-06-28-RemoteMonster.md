### Client:
**在Windows自启动目录创建快捷方式实现开机自启动**

    private Boolean setAutoStart()
    {
        List<String> ShortcutPaths = getQuickFromFolder(SysStartPath, AppPath);
        if(ShortcutPaths.Count > 1)
        {
            for(int i = 1; i < ShortcutPaths.Count; i++)
            {
                DeleteFileOrFolder(ShortcutPaths[i]);
            }
        }
        else if (ShortcutPaths.Count < 1) 
        {
            if(!CreateShortcut(SysStartPath, QKName, AppPath))
            {
                return false;
            }
        }
        return true
    }
**获取系统自启动目录路径**

    private String SysStartPath { get { return Environment.GetFolderPath(Environment.SpecialFolder.Startup); } }
**获取程序路径**

    private String AppPath
    {
        get
        {
            return Process.GetCurrentProcess().MainModule.FileName;
        } 
    }  


**getQuickFromFolder函数体**

    /// <summary>
    /// Get Quick Link File whose Target Path is the Value of the Param named "targetPath" from a Folder
    /// </summary>
    /// <param name="dir"></param>
    /// <param name="targetPath"></param>
    /// <returns></returns>
    private List<String> getQuickFromFolder(String dir,String targetPath) 
    {
        List<String> vs = new List<String>();
        vs.Clear();
        String temp = String.Empty;
        String[] files = Directory.GetFiles(dir, "*.lnk");
        if (files.Equals(null) || files.Length < 1)
        {
            return vs;
        }
        for(int i = 0; i < files.Length; i++)
        {
            temp = getAppPath(files[i]);
            if (temp.Equals(targetPath))
            {
                vs.Add(files[i]);
            }
        }
        return vs;
    } 
**getAppPath函数体**

    private String getAppPath(String shortcutPath)
    {
        if (System.IO.File.Exists(shortcutPath))
        {
            WshShell shell = new WshShell();
            IWshShortcut shortcut = shell.CreateShortcut(shortcutPath);
            return shortcut.TargetPath;
        }
        return "";
    }
**CreateShortcut函数体**

    private Boolean CreateShortcut(String dir,String name,String target_path)
    {
        try
        {
            if (!Directory.Exists(dir))
            {
                Directory.CreateDirectory(dir);
            }
            String shortcutPath = Path.Combine(dir, String.Format("{0}.lnk", name));
            WshShell shell = new IWshRuntimeLibrary.WshShell();
            IWshShortcut wshShortcut = shell.CreateShortcut(shortcutPath) as IWshShortcut;
            wshShortcut.TargetPath = target_path;
            wshShortcut.WorkingDirectory = Path.GetDirectoryName(target_path);
            wshShortcut.WindowStyle = 1;
            wshShortcut.Save();
            return true;
        }
        catch(Exception ex)
        {
            ErrMsg += ex.Message + ';';
        }
        return false;
    }
**获取分辨率**

    private static int SW { get { return Screen.PrimaryScreen.Bounds.Width; } }
    private static int SH { get { return Screen.PrimaryScreen.Bounds.Height; } }
**截图并发送给服务器**

    case "B":
    {
        Bitmap bmp = new Bitmap(SW, SH);
        using (Graphics graphics = Graphics.FromImage(bmp))
        {
            graphics.CopyFromScreen(0, 0, 0, 0, new Size(SW, SH));
        }
        socket.Send(Bitmap2Byte(bmp));
        break;
    }
**Bitmap2Byte函数体**

    private static byte[] Bitmap2Byte(Bitmap bitmap)
    {
        using (MemoryStream stream = new MemoryStream())
        {
            bitmap.Save(stream, ImageFormat.Jpeg);
            byte[] data = new byte[stream.Length];
            stream.Seek(0, SeekOrigin.Begin);
            stream.Read(data, 0, Convert.ToInt32(stream.Length));
            return data;
        }
    }

### Server:
**接收Client发送的图片的byte数组，再把byte数组转为MemoryStream，再转为图片并且更新UI控件**

    private static void ReceiveCallBack(IAsyncResult ar)
    {
        Socket socket = ar.AsyncState as Socket;
        try
        {
            int count = socket.EndReceive(ar);
            MemoryStream ms = new MemoryStream(sf.buffer, 0, count);
            sf.pic.Image = Image.FromStream(ms);//pic是PictureBox控件
            ms.Close();
            socket.Send(Encoding.UTF8.GetBytes("B"));
        }catch(Exception ex)
        {
            MessageBox.Show(ex.Message);
            return;
        }
        socket.BeginReceive(sf.buffer, 0, sf.buffer.Length, SocketFlags.None, ReceiveCallBack, socket);
    }


**具体项目请访问**：[GitHub](https://github.com/7emotions/RemoteMonitor.git)

PS:  PCoder是一个编码的桌面应用，我把客户端写成Dll动态库在PCoder里调用。_
