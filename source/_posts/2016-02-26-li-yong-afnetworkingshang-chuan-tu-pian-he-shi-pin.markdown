---
layout: post
title: "利用AFNetworking上传图片和视频"
date: 2016-02-26 09:15:43 +0800
comments: true
categories: iOS
---
##前言
之前尝试过用原生的上传代码，但是比较麻烦，利用afnetworking进行上传还是很方便的。

文件上传后台服务器必须要有相应的代码支持，我们后台使用java做的，首先ios客户端要把上传的图片或者视频转换成标准的文件流，然后传给后台，才能顺利实现上传功能。

<!--more-->
###上传图片

```
/**
 *  上传图片
 */
-(void)uploadPicture:(UIImage *)selectImg{
    //房源id：houseId 经纪人姓名:agentName flag(0未审核)文件:file
    NSMutableURLRequest *request = [[AFHTTPRequestSerializer serializer] multipartFormRequestWithMethod:@"POST" URLString:URL_UPLOADFILES parameters:nil constructingBodyWithBlock:^(id<AFMultipartFormData> formData)
                                    {
                                        NSData *imgData=UIImageJPEGRepresentation(selectImg, 1.0);
                                        [formData appendPartWithFileData:imgData name:@"file" fileName:@"ios.png" mimeType:@"image/jpeg"];
                                        
                                        NSData *data =[@"654321" dataUsingEncoding:NSUTF8StringEncoding];

                                        [formData appendPartWithFormData:data name:@"houseAppKey"];
                                        
                                        NSData *data1 =[_houseID dataUsingEncoding:NSUTF8StringEncoding];
                                        [formData appendPartWithFormData:data1 name:@"houseId"];
                                        
                                    }];
    
    AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    NSURLSessionUploadTask *uploadTask;
    
    uploadTask = [manager
                  uploadTaskWithStreamedRequest:request
                  progress:nil
                  completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
                      [[UIManager shareInstance] hide];
                      if (error) {
                          MYLog(@"Error: %@", error);
                      } else {
                          NSDictionary *dict=responseObject;
                          if(dict){
                              NSInteger result=[dict[@"code"] integerValue];
                              if(result==0){
                                  NSString *msg=dict[@"msg"];
                                  [[UIManager shareInstance] showSuccessWithMessage:msg duration:DURATIONTIME];
                              }else{
                                  [[UIManager shareInstance] showErrorWithMessage:@"上传失败" duration:DURATIONTIME];
                              }
                          }
                      }
                  }];
    [uploadTask resume];
}
```

###上传视频

```
#pragma mark
#pragma mark 上传视频
-(void)updateVedio:(NSData *)vedioData{
    
    
    //房源id：houseId 经纪人姓名:agentName flag(0未审核)文件:file
    NSMutableURLRequest *request = [[AFHTTPRequestSerializer serializer] multipartFormRequestWithMethod:@"POST" URLString:URL_UPLOADFILES parameters:nil constructingBodyWithBlock:^(id<AFMultipartFormData> formData)
                                    {
                                    
                                       [formData appendPartWithFileData:vedioData name:@"video1" fileName:@"video1.mov" mimeType:@"video/quicktime"];
                                        
                                        NSData *data =[@"654321" dataUsingEncoding:NSUTF8StringEncoding];
                                        
                                        [formData appendPartWithFormData:data name:@"houseAppKey"];
                                        
                                        NSData *data1 =[_houseID dataUsingEncoding:NSUTF8StringEncoding];
                                        [formData appendPartWithFormData:data1 name:@"houseId"];
                                        
                                    }];
    
    AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    NSURLSessionUploadTask *uploadTask;
    
    uploadTask = [manager
                  uploadTaskWithStreamedRequest:request
                  progress:nil
                  completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
                      [[UIManager shareInstance] hide];
                      if (error) {
                          MYLog(@"Error: %@", error);
                      } else {
                          NSDictionary *dict=responseObject;
                          if(dict){
                              NSInteger result=[dict[@"code"] integerValue];
                              if(result==0){
                                  NSString *msg=dict[@"msg"];
                                  [[UIManager shareInstance] showSuccessWithMessage:msg duration:DURATIONTIME];
                              }else{
                                  [[UIManager shareInstance] showErrorWithMessage:@"上传失败" duration:DURATIONTIME];
                              }
                          }
                      }
                  }];
    [uploadTask resume];
}
```

###图片多选
如果想要一次上传多张图片的话，可能会用到 
 [AssetsPickerController](https://github.com/chiunam/CTAssetsPickerController)  
 这个第三方库
 
 我在代码中用法如下:
 
 ```
 -(void)takeVedios:(UIButton *)btn{
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status){
        dispatch_async(dispatch_get_main_queue(), ^{
            
            // init picker
            CTAssetsPickerController *picker = [[CTAssetsPickerController alloc] init];
            // set delegate
            picker.delegate = self;
            // Optionally present picker as a form sheet on iPad
            if (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPad)
                picker.modalPresentationStyle = UIModalPresentationFormSheet;
            
            // present picker
            [self presentViewController:picker animated:YES completion:nil];
        });
    }];
}

 ```
 
 
###补充
上传的时候最好有进度条进行提示，这是比较好的用户体验，AFNetworking官方也给出了上传下载的进度条实例，我这块之前没有做出很好的效果，以后有时间了再研究一下。

我用的AF的版本号是2.6的可能和最新版本的AF上传的代码有点区别，进度条的展示可能也有点区别，AFNetworking中的进度条用了NSProcess来显示，NSProcess可以用 KVO来监控实现，

代码片段:

```
/**
 *  上传临时图片到服务器
 *
 *  @param selectImg 要传入的图片
 *  @param callback  回调函数
 */
-(void)uploadToServerTEMP:(UIImage *)selectImg
      UploadTEMPFileBlock:(UploadTEMPFileBlock)callback{
    
    
    NSMutableURLRequest *request = [[AFHTTPRequestSerializer serializer] multipartFormRequestWithMethod:@"POST" URLString:URL_UPLOADTEMP parameters:nil constructingBodyWithBlock:^(id<AFMultipartFormData> formData)
                                    {
                                        NSData *imgData=UIImageJPEGRepresentation(selectImg, 1.0);
                                        [formData appendPartWithFileData:imgData name:@"Filedata" fileName:@"ios.png" mimeType:@"image/jpeg"];
                                    }];
    
    AFURLSessionManager *manager = [[AFURLSessionManager alloc] initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    NSURLSessionUploadTask *uploadTask;
    
    NSProgress *progress = nil;
    
    uploadTask = [manager
                  uploadTaskWithStreamedRequest:request
                  progress:&progress
                  completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
                      if (error) {
                          MYLog(@"Error: %@", error);
                          callback(nil,error);
                      } else {
                          NSDictionary *dict=responseObject;
                          if(dict){
                              callback(dict,nil);
                          }
                          //[progress removeObserver:self forKeyPath:@"fractionCompleted"];
                      }
                  }];
    
    [progress addObserver:self
               forKeyPath:@"fractionCompleted"
                  options:NSKeyValueObservingOptionNew
                  context:NULL];
    [uploadTask resume];
    
}

- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    //[super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    
    if ([keyPath isEqualToString:@"fractionCompleted"] && [object isKindOfClass:[NSProgress class]]) {
        NSProgress *progress = (NSProgress *)object;
        NSLog(@"Progress is %f", progress.fractionCompleted);
      
        [_processView setProgress:progress.fractionCompleted animated:YES];
        
         NSLog(@"progress.totalUnitCount:%lld;progress.completedUnitCount=%lld",progress.totalUnitCount,progress.completedUnitCount);
    }
}
```

经过检验，进度条显示有点问题，可能不是实时显示，进度条显示的不太准确。需要后期优化。


