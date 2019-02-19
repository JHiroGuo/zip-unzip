# zip-unzip
基于C语言 跨平台zip/unzip


## zip
(1) 传统用法，从现有文件创建zip文件    HZIP hz = CreateZip("c:\\simple1.zip",0);    ZipAdd(hz,"znsimple.bmp", "c:\\simple.bmp");    ZipAdd(hz,"znsimple.txt", "c:\\simple.txt");    CloseZip(hz);

 (2) 内存使用，从各种来源创建一个自动分配的基于内存的zip文件    HZIP hz = CreateZip(0,100000, 0);    // adding a conventional file...    ZipAdd(hz,"src1.txt",  "c:\\src1.txt");    // adding something from memory...    char buf[1000];
    for (int i=0; i<1000; i++) 
    	buf[i]=(char)(i&0x7F);    ZipAdd(hz,"file.dat",  buf,1000);    // adding something from a pipe...    HANDLE hread,hwrite; 
    CreatePipe(&hread,&hwrite,NULL,0);    HANDLE hthread = CreateThread(0,0,ThreadFunc,(void*)hwrite,0,0);    ZipAdd(hz,"unz3.dat",  hread,1000);  // the '1000' is optional.    WaitForSingleObject(hthread,INFINITE);    CloseHandle(hthread); 
    CloseHandle(hread);    // and now that the zip is created, let's do something with it:    void *zbuf; unsigned long zlen; 
    ZipGetMemory(hz,&zbuf,&zlen);    HANDLE hfz = CreateFile("test2.zip",GENERIC_WRITE,0,0,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL,0);    LDWORD writ; 
    
    WriteFile(hfz,zbuf,zlen,&writ,NULL);    CloseHandle(hfz);    CloseZip(hz);(3) 句柄用于文件句柄和管道    HANDLE hzread,hzwrite; 
    
    CreatePipe(&hzread,&hzwrite,0,0);    HANDLE hthread = CreateThread(0,0,ZipReceiverThread,(void*)hzread,0,0);    HZIP hz = CreateZipHandle(hzwrite,0);    // ... add to it    CloseZip(hz);    CloseHandle(hzwrite);    WaitForSingleObject(hthread,INFINITE);    CloseHandle(hthread);## unzip

(1) 传统方法

```SetCurrentDirectory("c:\\docs\\stuff");HZIP hz = OpenZip("c:\\stuff.zip",0);ZIPENTRY ze; 
GetZipItem(hz,-1,&ze); 
int numitems=ze.index;for (int i=0; i<numitems; i++){ 
  GetZipItem(hz,i,&ze);  UnzipItem(hz,i,ze.name);}CloseZip(hz);

```
(2) 特殊需求```HRSRC hrsrc = FindResource(hInstance,MAKEINTRESOURCE(1),RT_RCDATA);HANDLE hglob = LoadResource(hInstance,hrsrc);void *zipbuf=LockResource(hglob);unsigned int ziplen=SizeofResource(hInstance,hrsrc);HZIP hz = OpenZip(zipbuf, ziplen, 0); // - unzip to a membuffer -ZIPENTRY ze; int i; 

FindZipItem(hz,"file.dat",true,&i,&ze);char *ibuf = new char[ze.unc_size];UnzipItem(hz,i, ibuf, ze.unc_size);delete[] ibuf;  // - unzip to a fixed membuff -ZIPENTRY ze; int i; 

FindZipItem(hz,"file.dat",true,&i,&ze);char ibuf[1024]; ZRESULT zr=ZR_MORE; unsigned long totsize=0;while (zr==ZR_MORE){ 
  zr = UnzipItem(hz,i, ibuf,1024);  unsigned long bufsize=1024; 
  if (zr==ZR_OK) 
    bufsize=ze.unc_size-totsize;  totsize+=bufsize;}  // - unzip to a pipe -HANDLE hwrite;

HANDLE hthread=CreateWavReaderThread(&hwrite);int i; ZIPENTRY ze; 

FindZipItem(hz,"sound.wav",true,&i,&ze);UnzipItemHandle(hz,i, hwrite);CloseHandle(hwrite);WaitForSingleObject(hthread,INFINITE);CloseHandle(hwrite); 

CloseHandle(hthread);//   - finished -CloseZip(hz);// note: no need to free resources obtained through Find/Load/LockResource```(3) 获取zip的item

```SetCurrentDirectory("c:\\docs\\pipedzipstuff");HANDLE hread,hwrite; CreatePipe(&hread,&hwrite,0,0);CreateZipWriterThread(hwrite);HZIP hz = OpenZipHandle(hread,0);for (int i=0; ; i++){ ZIPENTRY ze;  ZRESULT zr=GetZipItem(hz,i,&ze); 
  if (zr!=ZR_OK) 
  		break; // no more  UnzipItem(hz,i, ze.name);}CloseZip(hz);```