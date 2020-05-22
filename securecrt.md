.命令sz、rz的使用方法：

rz中的r意为received（接收），输入rz时、意为服务器接收文件，既将文件从本地上传到服务器。
sz中的s意为send（发送），输入sz时、意为服务器要发送文件，既从服务器发送文件到本地，或是说本地从服务器上下载文件。
注：不论是send还是received，动作都是在服务器上发起的。

上传文件只需在shell终端仿真器中输入命令"rz",即可从弹出的对话框中选择本地磁盘上的文件，利用Zmodem上传到服务器当前路径下。

　　下载文件只需在shell终端仿真器中输入命令"sz 文件名",即可利用Zmodem将文件下载到本地某目录下。

　　通过"File Transfer"可以修改下载到本地的默认路径。设置默认目录：options-->session options-->file transfer.
