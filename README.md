# valetudo_voice_pack_arnold_standard
This is a voice pack of all the Dreame D10S Plus messages repeated in the synthesized voice of Arnold Schwarzenegger. These messages were created with an online voice synthesizer.

I just installed Valetudo on my D10S Plus and then I found this GitHub repo for installing a voice pack: https://gist.github.com/xmesaj2/33af535ee9eb37c947135042c4ebb4c0

It looked pretty straight forward but wasn't for my vacuum. I compared the numbered ogg files in the repo with the ogg files on my robot and there were differences on both sides.
Because of this I created my own set of ogg files using fakeyou: https://fakeyou.com/tts They have an excellent voice model of Arnold Schwarzenegger, who's one of my personal heros. If I could have him say things as he cleans my house, that'd be outstanding!

I "borrowed" the following sequence from xmesaj2's repo and tweaked it slightly for my process. I never used ngrok but it worked really well from a quick `docker run`.

1. Root
2. Install Valetudo
3. Backup files with SCP and to make a list of all of them
```console 
scp -i key.id_rsa root@192.168.1.101:/audio/EN/* backup/
```
4. Download/create your .wav files, save as `0.wav` `1.wav` etc.
https://github.com/Findus23/voice_pack_dreame/blob/main/sound_list.csv use this for reference which file is which sound to avoid listening to original files to find out
5. Normalize WAV and convert to OGG. 
I used WSL Ubuntu 20.04 on Win11 (install `vorbis-tools`, `ffmpeg`)
```console 
mkdir ogg
cp backup/* ogg/
filenames=("7" "8" "10" "11" "12" "13" "18" "30" "41" "45" "110" "149" "151")
for filename in ${filenames[@]}; do ffmpeg -i "${filename}.wav" -filter:a loudnorm=I=-14:LRA=1:dual_mono=true:tp=-1 "${filename}_n.wav" && oggenc "${filename}_n.wav" --output "${filename}.ogg" --bitrate 100 --resample 16000; done
mv *.ogg ogg/
cd ogg
tar czf ../arnold_voice.tar.gz *.ogg
md5sum ../arnold_voice.tar.gz
```
6. Serve files. I use (PowerShell/CMD, not WSL!!) https://dashboard.ngrok.com/get-started/setup) and python 3
```console 
python -m http.server 8000 --bind 0.0.0.0
docker run --net=host -it ngrok/ngrok http 8000
```
7. Open ngrok url in web browser, right click `arnold_voice.tar.gz` "copy link"
8. Go to Valetudo eg. http://192.168.1.101/#/robot/settings/misc language ARNOLD, hash from md5sum command -- I think it was e80ac063e0e9e930cad2f9d7b2d7648a

-- if your robot has access to github you might be able to just use it as the 
