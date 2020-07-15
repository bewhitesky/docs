# 文件下载

```java
	public void download(HttpSession session, HttpServletResponse response) {
		FileInputStream fileInputStream = null;
		ServletOutputStream servletOutputStream = null;
		String basePath = session.getServletContext().getRealPath("/"); // 获取基本路径
		BaseInfo filedown = baseInfoService.selectBase();
        //路径和文件名建议存放在数据库
		String fileurl = filedown.getPath() +"/" + filedown.getFilename();
		String fileName = filedown.getFilename();
		try {
			response.setContentType("application/vnd.ms-excel;");
			response.setHeader("Content-disposition", "attachment; filename=" + fileName + ";charset=UTF-8");
			// 将本地文件装载到内存
			fileInputStream = new FileInputStream(basePath + fileurl);
			// 实例化输出流
			servletOutputStream = response.getOutputStream();
			byte[] buff = new byte[20480];
			int bytesRead;
			// 每次尝试读取buff.length长字节，直到读完、bytesRead为-1
			while ((bytesRead = fileInputStream.read(buff, 0, buff.length)) != -1) {
				// 每次写bytesRead长字节
				servletOutputStream.write(buff, 0, bytesRead);
			}
			// 刷新缓冲区
			servletOutputStream.flush();
		} catch (IOException e) {
			throw new BusinessException("下载模板失败");
		} finally {
			if (fileInputStream != null) {
				try {
					fileInputStream.close();
				} catch (IOException e) {
					throw new BusinessException("关闭文件流失败");
				}
			}
			if (servletOutputStream != null) {
				try {
					servletOutputStream.close();
				} catch (IOException e) {
					throw new BusinessException("关闭文件流失败");
				}
			}
		}
	}
```