这是一个轻便的录音activity, 
请看这里: https://github.com/adrielcafe/AndroidAudioRecorder

在原项目基础上做了一点改进：
1. 点击“√”（保存）时，将录音得到的wav文件转换成mp3格式。在录音比较长的情况下，转换要花费一点点时间，
    因此添加了waiting的动画。
2. 点击取消或者返回，则会删除掉wav文件（原版本无论结果如何，都将生成wav文件）。
3. 稍微改变了使用方法：要求传入文件夹的路径（原版本传入的是目标文件的路径: xxx/xxx.wav）
    当保存成功时，传回mp3文件的路径（String），保存在onActivityResult方法的data参数中，名为'mp3_file_path'
4. 支持后台录音

用法：
1. 导入依赖

allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
}

dependencies {
    implementation 'com.github.Chen-Xuming:AndroidAudioRecorder:0.3.1'
}

2. 添加权限
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WAKE_LOCK" />

注意：其中前两个需要动态申请！

3. 使用

调用以下方法就可以进入录音界面:

AndroidAudioRecorder.with(this)
                // Required
                .setFilePath(external_music_files_dir)              // ！！注意：这里和原版本不一样
                                                                    //需要存储到的文件夹的路径（请确保文件夹存在且路径正确）
                                                                    
                .setColor(ContextCompat.getColor(this, R.color.recorder_bg))    // 设置录音界面主题色
                .setRequestCode(REQUEST_RECORD_AUDIO)                           // 设置请求码（REQUEST_RECORD_AUDIO是用户自己定义的）

                // Optional
                .setSource(AudioSource.MIC)                         // 录音源：麦克风             
                .setChannel(AudioChannel.STEREO)                    // 双声道
                .setSampleRate(AudioSampleRate.HZ_48000)            // 码率（这里产生的wav临时文件是48000Hz, 转换后的mp3文件是44100Hz）
                .setAutoStart(false)                                // 是否自动录音（进入录音界面后自动开始）
                .setKeepDisplayOn(true)                             // I don't know.

                // Start recording
                .record();
                
退出录音界面后可以在Activity类的onActivityResult得到一些返回结果：
例如：
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQUEST_RECORD_AUDIO) {
            if (resultCode == RESULT_OK) {
                String mp3 = data.getStringExtra("mp3_file_path");
                Toast.makeText(this, "The .mp3 file path:\n" + mp3, Toast.LENGTH_LONG).show();
            } else if (resultCode == RESULT_CANCELED) {
                Toast.makeText(this, "Cancel", Toast.LENGTH_SHORT).show();
            }else if(resultCode == 2){
                Toast.makeText(this, "Save failed", Toast.LENGTH_SHORT).show();
            }
        }
    }
从上面代码可以看到，录音界面退出后的返回结果如下：
RESULT_OK:          保存成功， 并且可以在data参数里获得文件路径：mp3_file_path
RESULT_CANCELED:    取消（点击‘X’或者点击手机的返回键）
2：                 点击了‘√’但是生成mp3文件失败

具体的demo可以参照源代码！
