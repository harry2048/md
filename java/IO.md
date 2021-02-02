## DataInputStream数据过大时读不全

> https://www.cnblogs.com/chengjun/p/9013856.html

```java
public byte[] readValue(Map<String,Object> map){
    int le = 0;
    int len = din.read();
    le++;
    byte[] ret = null;
    if (len < 255){
        ret = new byte[len];
        din.readFully(ret);
        String msg = new String(ret);
        System.out.println("数据为->" + msg);
        return ret;
    }
    
    // 多次读取
    ByteArrayOutputStream bou = new ByteArrayOutputStream();
    byte[] buf = new byte[255];
    din.readFully(buf);
    bou.write(buf);
    while(true){
        len = din.read();
        le++;
        if (len == 255) {
            din.readFully(buf);
            bou.write(buf);    
        }else{
            byte[] lastPart = new byte[len];
            din.readFully(lastPart);
            bou.write(lastPart);
            break;
        }
    }
    
    return bou.toByteArray();
}
```

## jboss版上传

```java
import java.on.*;
import org.jboss.resteasy.plugins.providers.multipart.InputPart;
import org.springframework.util.FileCopyUtils;
public void upload(String targetPath,Map<String,List<InputPart>> formDataMap){
  byte]] template=null;
  File targetfile = new File(targetPath);
  if(!targetfile.exists() || !targetfile.isDirectory()){
    targetfile.mkdirs();
  }

  //写入文件
  List<InputPart> files=formDataMap.get("fileUpload");//上传框的id
  if(files != null){
    for(InputPart inputPart: files){
      InputStream inputStream=null;
      FileOutputStream os=null;
      try{
          //获取上传文件
        inputStream=inputPart.getBody(InputStream.class,null);
        template=FileCopyUtils.copyToByteArray(inputStream);
        
        File outfile =new File(targetPath+File.separator+fileName) ;
        os=new FileOutputStream(outfile,false);
        os.write(template,0,template.length);
        os.flush();
        os.close();
        inputStream.close();
      }catch(Exception e){
        throw e;
      }finally{
        try{
           if(os!=null){
             os.close();
           }
           if(inputStream!=null){
             inputStream.close();
           }
        }catch(Exception e){
          throw e;
        }
      }
    }
  }
}
```

## 下载

> 先开后关，只需要关最外层的流。
>
> File.separator    是  \

```java
import java.io.*;
import org.springframework.util.FileCopyUtils;

public void download(String targetPath,HttpServletRequest request,HttpServletResponse){
  BufferedInputStream inputStrram=null;
  BufferedOutputStream outputStream=null;
  FileInputStream in=null;
  try{
      String fileName = request.getParameter('fileName');
      byte ]] template =null;
      File file = new File(targetPath+File.separater+fileName);
      
      response.reset();
      response.setContentType("application/vnd.ms-excel;charset=utf-8");
      response.setHeader("Content-Disposition","attachment;filename="+new String(fileName.getBytes(),"iso-8859-1"));
      
      in=new FileInputStream(file);
      inputStream=new BufferedInput(in);
      os=new BufferedOutputStream(response.getOutputStream());
      template=FileCopyUtils.copyToByteArray(inputStream);
      os.write(template,0,template.length);
      os.flush();
      os.close();
      
      inputStream.close();
      in.close();
      os=null;
      in=null;
      inputStream=null;
      
  }catch(Exception e){
      e.print();
  }finally{
      try{
         if(os!=null){
            os.close();
         }
         if(inputStream!=null){
            inputStream.close();
         }
      }catch(Exception e){
          e.print..();
      }
 }
}
```

## SpringBoot + ajax 上传

> ajax的文件上传，controller接收的时候，参数为Multipart file，名称必须相同file=file

```js
$(function () {
  $('.myFileUpload').change(function(){
     var formdata=new FormData()
     formdata.append('file',$("#file")[0].files[0])
     $.ajax({
         url:"http://localhost:8080/import/idPositive",
         data:formdata,
         dataType:'json',//服务器返回json格式数据
         type:'post',//HTTP请求类型
        contentType:false,
        processData:false,    //防止自动转换数据格式
        success:function(res){
             if(res.success){
                
             }
         },
     });
 });
})
```

```properties
#Spring boot1.0的设置如下：
#1 spring.http.multipart.max-file-size=100mb
#2 spring.http.multipart.max-request-size=1000mb

#Spring boot2.0的设置如下：
#1 #设置上传APP的大小限制
#2 spring.servlet.multipart.max-file-size=100Mb
#3 spring.servlet.multipart.max-request-size=100Mb
```



