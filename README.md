# ffserver_rtl-sdr

使用openwrt做网络收音机的笔记：  

1、openwrt安装ffserver、ffmpeg、rtl_sdr  

2、创建/etc/ffserver.conf  
HttpPort 1234   
HttpBindAddress 0.0.0.0   
MaxHTTPConnections 200   
MaxClients 100  
MaxBandwidth 2000  
CustomLog -  
#NoDaemon  
<Feed feed2.ffm>  
        File /tmp/feed2.ffm  
        FileMaxSize 10M  
        ACL allow 127.0.0.1  
</Feed>   
<Stream demoAudio.mp3>  
        Format mp2  
        Feed feed2.ffm  
        AudioBitRate 320  
        AudioChannels 2  
        AudioSampleRate 44100  
        AVOptionAudio flags +global_header  
        NoVideo  
</Stream>  

3、创建start_mms.sh  
#!/bin/ash  
ffserver -f /etc/config/ffserver.conf &  
rtl_fm -f 99.3M -M fm -s 320k -o 4 -A fast -r 44100 -l 0 -E deemp -g 49.6 - | ffmpeg -f s16le -ac 1 -i pipe:0 -acodec libmp3lame -vol 256 http://127.0.0.1:1234/feed2.ffm &  

4、创建stop_mms.sh  
#!/bin/ash  
ps | grep rtl_fm | grep -v "grep" | awk '{print $1}' | xargs kill -9  
ps | grep ffserver | grep -v "grep" | awk '{print $1}' | xargs kill -9  

5、启动：start_mms.sh，停止：stop_mms.sh  
