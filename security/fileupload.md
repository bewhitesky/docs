# 上传文件校验
```java
        @Resource
        private FileupService fileupService;
    
        @ResponseBody
        @RequestMapping(value = "/upload" ,method = RequestMethod.POST)
        public String upload(@RequestParam("file") MultipartFile file) throws IOException {
            //从数据库取地址路径和文件大小限制
            FileDto files = fileupService.selectFileup();
            int size = Integer.valueOf(files.getSize());
            String filePath = files.getPath();
    
            if (file.getSize()>size) {
                return "file size error";
            }
    
            String fileName = file.getOriginalFilename();
            String suffixName = fileName.substring(fileName.lastIndexOf(".")+1);
            if (FileUtil.checkFile(suffixName)){
                String path = filePath + fileName;
                File dest = new File(path);
    
                if (!dest.getParentFile().exists()) {
                    dest.getParentFile().mkdirs();
                }
                file.transferTo(dest);
                return "upload success";
            }
    
            return "upload failure";
        }

```
* 工具类
```java
public class FileUtil {
    public static String IMG_TYPE_PNG = "PNG";
    public static String IMG_TYPE_JPG = "JPG";
    public static String IMG_TYPE_JPEG = "JPEG";
    public static String IMG_TYPE_DMG = "zip";

    public static Boolean checkFile(String filename){

        if(IMG_TYPE_PNG.equalsIgnoreCase(filename)||
                IMG_TYPE_JPG.equalsIgnoreCase(filename)||
                IMG_TYPE_JPEG.equalsIgnoreCase(filename)||
                IMG_TYPE_DMG.equalsIgnoreCase(filename)
        ){
            return true;
        }else {
            return false;
        }
    }
}
```