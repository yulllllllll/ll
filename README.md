# ll// ==用户脚本==
// @name youtube中英双语字幕
// @namespace https://github.com/wlpha/Youtube-Automatic-Translation-V2
// @match *://www.youtube.com/watch?v=*
// @match *://www.youtube.com
// @match *://www.youtube.com/*
// @author wlpha
// @版本 0.2.2
// @run-at 文档开始
// @description 油管自动跳广告，机翻中英双语字幕，视频下载，srt 字幕下载，不常看greasyfork，有问题请发邮件：vsq_kuangqi@qq.com
// @homepage https://greasyfork.org/zh-CN/scripts/406994-youtube%E4%B8%AD%E8%8B%B1%E5%8F%8C%E8%AF%AD%E5%AD% 97%E5%B9%95
// ==/用户脚本==


（功能（） {
    var captionWidget = null;
    var captionSubWidget = null;
    var captionConrtol_1 = null;
    var captionConrtol_2 = null;
    var captionUrl_1 = null;
    var captionUrl_2 = null;
    var visiableCaption = false;
    var captionTranslateContent_1 = null;
    var captionTranslateContent_2 = null;
    var hiddenCaptionsTimer_1 = null;
    var hiddenCaptionsTimer_2 = null;
    var captionDelayLoadTime = 1;
    var checkTranslateCaptionTimer = null;
    var hasFetchTranslationButton = null;
    var hasSubtitleLabel = null;
    var saveEnSubtitleButton = null;
    var saveEnSubtitleButton = null;
    var downloadVideoList = null;
    var saveDownloadVideoButton = null;

    var captionFontBottomPosition = window.localStorage.getItem('double-translation-font-position');
    captionFontBottomPosition = captionFontBottomPosition ? captionFontBottomPosition : '60px';
    var captionFontMiniBottomPosition = window.localStorage.getItem('double-translation-font-mini-position');
    captionFontMiniBottomPosition = captionFontMiniBottomPosition ? captionFontMiniBottomPosition : '10px';
    var captionFontSize = window.localStorage.getItem('double-translation-font-size') 
    标题字体大小 = 标题字体大小？标题字体大小：'2.0em';
    var captionFontMiniSize = window.localStorage.getItem('double-translation-font-mini-size')
    captionFontMiniSize = captionFontMiniSize ? captionFontMiniSize :'1.4em';
    var captionFontPadding = '0px 6px';
    var captionFontColor = '#ffffff';
    var captionFontTextShadow = '0 0 2px #000';
    var captionFontTextStroke = '#00000f0 3px';
    
    window.doubleTranlationPluginState = {};

    函数 GenerateCaptionsUrl(toLang)
    {
        var url = window.ytInitialPlayerResponse.captions.playerCaptionsTracklistRenderer.captionTracks[0].baseUrl;
        /*
        if(url.indexOf('&lang=zh-Hant') !== -1) 
        {
            url = url.replace('&lang=zh-Hant', '&lang=zh-Hans');
        }
        */
        返回网址 + '&fmt=json3&xorb=2&xobt=3&xovt=3&tlang=' + toLang;
    }
    函数 HasTranslateCaption()
    {
        尝试
        {
            window.ytInitialPlayerResponse.captions.playerCaptionsTracklistRenderer.captionTracks;
            返回真；
        }抓住（错误）
        {
            返回假；
        }
    }

    函数 GetCurrentVideoPlayerUrls()
    {
        var url = window.location.href + '&pbj=1';
        变量标头 = {
                    'x-youtube-client-name': 1,
                    “x-youtube-client-version”：window.ytcfg.data_.INNERTUBE_CONTEXT_CLIENT_VERSION，
        };
        如果（window.ytcfg.data_.ID_TOKEN）{
          headers['x-youtube-identity-token'] = window.ytcfg.data_.ID_TOKEN
        };
      
        获取（网址， 
            {
                标题：标题
            })
        .then（功能（响应） 
        {
            response.json().then(function(response)
            {
                var videoListInfo = [];

                if(typeof response === 'string')
                {
                    响应 = JSON.parse(响应);
                }
                
                for(var i=0; i<response.length; i++)
                {
                    // 调试器
                    var item = response[i];
                    如果（item.playerResponse）
                    {
                        var data = item.playerResponse;
                        if(data && data.streamingData && data.streamingData.adaptiveFormats) 
                        {
                            var dataList = data.streamingData.adaptiveFormats;
                            for(var x=0; x<dataList.length; x++)
                            {
                                var dataItem = dataList[x];

                                如果（！dataItem.url） 
                                {
                                    如果（数据项.signatureCipher） 
                                    {
                                        var dict = DecodeURIParamToDict(dataItem.signatureCipher);
                                        dataItem.url = dict.url + '&' + dict.sp + '=' + encodeURIComponent(DecryptBySignatureCipher(dict.s));
                                    } 别的 
                                    {
                                        继续;
                                    }
                                }

                                var 标签 = '';

                                if(dataItem.mimeType.indexOf('video/') > -1) 
                                {
                                    var videoType = dataItem.mimeType.match(/(\w+)\/(.*?);/i);
                                    var videoCodec = dataItem.mimeType.match(/codecs="(.*?)"/i)[1];
                                    label = '[视频]' + dataItem.qualityLabel + ';' + 视频类型[1] + '/' + 视频类型[2] + ';' + 视频编解码器；
      
                                } 别的 
                                {
                                    var videoType = dataItem.mimeType.match(/(\w+)\/(.*?);/i);
                                    var videoCodec = dataItem.mimeType.match(/codecs="(.*?)"/i)[1];
                                    label = '[未知]' + videoType[1] + '/' + videoType[2] + ';' + 视频编解码器；
                                }

                                var itemInfo = {
                                    '标签'：标签，
                                    'url'：dataItem.url
                                }
                                控制台日志（数据项）；
                                videoListInfo.push(itemInfo);
                                
                            }
                        }
                    }
                }
                
                // 移除原来的
                下载VideoList.innerHTML = '';
                var option = document.createElement('option');
                option.setAttribute('value', 'none');

                如果（videoListInfo.length > 0） 
                {
                    option.innerText = '选择视频下载';
                } 别的 
                {
                    option.innerText = '获取不到链接';
                }
                
                下载VideoList.appendChild(option);
                // 添加到列表
                无功指数 = 0;
                for(var i=0; i<videoListInfo.length; i++)
                {
                    var itemInfo = videoListInfo[i];
                    option = document.createElement('option');
                    option.setAttribute('value', itemInfo.url);
                    option.innerText = (++index) + '.' + itemInfo.label;
                    下载VideoList.appendChild(option);
                }

                downloadVideoList.style.display = 'inline-block';
                saveDownloadVideoButton.style.display = 'inline-block';
                
            });
            
        }).catch（函数（错误）
        {

        });
    }

    函数 GetTranslateCaptionsUrl() 
    {
    
        var videoId = window.ytInitialPlayerResponse.videoDetails.videoId;
        var videoTitle = window.ytInitialPlayerResponse.videoDetails.title;
        var videoAuthor = window.ytInitialPlayerResponse.videoDetails.author;
        var videoViewCount = window.ytInitialPlayerResponse.videoDetails.viewCount;

        var captionTracks = window.ytInitialPlayerResponse.captions.playerCaptionsTracklistRenderer.captionTracks;
        var hasZhHans = false;
        var hasZhHant = false;
        var hasEn = false;
        var defaultLang = null;
        var selectLang = null;

        尝试
        {


            for(var i=0; i< captionTracks.length; i++)
            {
                var item = captionTracks[i];
                开关（项目。语言代码）
                {
                    案例'zh-Hans'：
                        hasZhHans = true;
                        休息;
                    案例'zh-Hant'：
                        hasZhHant = true;
                        休息;
                    案例“en”：
                        hasEn = true;
                        休息;
                    默认：
                        defaultLang = item.languageCode;
                        休息;
                }
            }

            if(hasZhHans)
            {
                selectLang = 'zh-Hans';
            }
            否则如果（hasZhHant）
            {
                selectLang = 'zh-Hant';
            }
            否则如果（hasEn）
            {
                selectLang = 'en';
            }
            否则 if(defaultLang !== null)
            {
                selectLang = defaultLang;
            }

            // 没有可用的翻译字幕
            if(selectLang === null)
            {
                console.log('没有可用的翻译字幕');
                返回;
            }

            // 双语字幕
            captionUrl_1 = GenerateCaptionsUrl('zh-Hans');
            captionUrl_2 = GenerateCaptionsUrl('en');

            console.log('-----------------youtube自动切换中文字幕(油猴插件)----------------- -----');
            console.log('视频标题：' + videoTitle);
            console.log('视频作者：' + videoAuthor);
            console.log('视频ID：' + videoId);
            console.log('播放数：' + videoViewCount);
            console.log('中文字幕：' + captionUrl_1);
            console.log('中文字幕：' + captionUrl_2);
        }catch(e)
        {

        }

    }
  
    函数 ResetTranslateCaptionsUrl()
    {
        captionUrl_1 = 空；
        captionUrl_2 = 空；
    }

    函数 FetchTranslateCaptionsContent()
    {
        visiableCaption = 假；
        captionTranslateContent_1 = null;
        captionTranslateContent_2 = null;

        如果 (captionUrl_1 !== null) {
            获取（标题Url_1）
            .then（功能（响应）
            {
                response.json().then(function(response)
                {
                    visiableCaption = 真；
                    captionTranslateContent_1 = response.events;

                    if(hasFetchTranslationButton !== null) {
                        hasFetchTranslationButton.innerText = '正常';
                    }

                    // 显示字幕取消按钮
                    var subtitleContent = GenerateSRTFromZhhans();
                    var fileContent = 'data:text/plain;charset=utf-8,' + encodeURIComponent(subtitleContent);
                    var filename = '[中字]' +window.ytInitialPlayerResponse.videoDetails.title + '.srt';
                    saveZhhansSubtitleButton.setAttribute('href', fileContent);
                    saveZhhansSubtitleButton.setAttribute('下载', 文件名);

                    hasSubtitleLabel.style.display = 'inline-block';
                    saveZhhansSubtitleButton.style.display = 'inline-block';

                    console.log('获取到字幕【中文】：成功');
                    
                }).catch（函数（错误）
                {
                    captionTranslateContent_1 = null;

                    if(hasFetchTranslationButton !== null) {
                        hasFetchTranslationButton.innerText = '失败';
                    }
                    console.log('获取到字幕【中文】：失败2');
                });

            }).catch（函数（错误）
            {
                captionTranslateContent_1 = null;

                if(hasFetchTranslationButton !== null) {
                    hasFetchTranslationButton.innerText = '失败';
                }
                console.log('获取到字幕【中文】：失败');
            });
        }

        如果 (captionUrl_2 !== null) {
            获取（标题Url_2）
            .then（功能（响应）
            {
                response.json().then(function(response)
                {
                    visiableCaption = 真；
                    captionTranslateContent_2 = response.events;
                    // 显示字幕取消按钮
                    var subtitleContent = GenerateSRTFromEn();
                    var fileContent = 'data:text/plain;charset=utf-8,' + encodeURIComponent(subtitleContent);
                    var filename = '[英字]' + window.ytInitialPlayerResponse.videoDetails.title + '.srt';
                    saveEnSubtitleButton.setAttribute('href', fileContent);
                    saveEnSubtitleButton.setAttribute('下载', 文件名);

                    hasSubtitleLabel.style.display = 'inline-block';
                    saveEnSubtitleButton.style.display = 'inline-block';

                    console.log('获取到字幕【中文】：成功');

                }).catch（函数（错误）
                {
                    控制台日志（错误）；
                    captionTranslateContent_2 = null;
                    console.log('获取到字幕【中文】：失败2');
                });

            }).catch（函数（错误）
            {
                captionTranslateContent_2 = null;
                console.log('获取到字幕【中文】：失败');
            });
        }

    }
    
    函数 CreateCaptionControl()
    {
        var mainPlayer = document.querySelector('#ytd-player .html5-video-player');
        如果（主播放器 === 空） 
        {
            返回假；
        }
        

        captionWidget = document.querySelector('#captionWidget');
        captionSubWidget = document.querySelector('#captionSubWidget');
        captionConrtol_1 = document.querySelector('#captionConrtol_1');
        captionConrtol_2 = document.querySelector('#captionConrtol_2');

        如果 (captionConrtol_1 !== null) {
            captionConrtol_1.style.display = '无';
        }
        如果 (captionConrtol_2 !== null) {
            captionConrtol_2.style.display = '无';
        }

        if(captionWidget !== null)
        {
            返回真；
        } 别的 
        {
            captionWidget = document.createElement('div');
            captionSubWidget = document.createElement('div');
            captionConrtol_1 = document.createElement('p');
            captionConrtol_2 = document.createElement('p');

            captionWidget.id = 'captionWidget';
            captionSubWidget.id = 'captionSubWidget';
            captionConrtol_1.id = 'captionConrtol_1';
            captionConrtol_2.id = 'captionConrtol_2';

   
            mainPlayer.parentElement.style.height = '100%';

            captionWidget.style.pointerEvents = 'none';
            captionWidget.style.position = '绝对';
            captionWidget.style.zIndex = 99999999;
            captionWidget.style.width = '100%';
            captionWidget.style.height = '100%';
            

            captionSubWidget.style.position = '绝对';
            captionSubWidget.style.width = '100%';
            captionSubWidget.style.height = 'fit-content';
            captionSubWidget.style.bottom = captionFontBottomPosition;
            
            captionConrtol_1.style.width = captionConrtol_2.style.width = 'fit-content';
            captionConrtol_1.style.width = captionConrtol_2.style.width = '-moz-fit-content';
            captionConrtol_1.style.margin = captionConrtol_2.style.margin = '0 自动';
            captionConrtol_1.style.padding = captionConrtol_2.style.padding = captionFontPadding;
            captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontSize;
            captionConrtol_1.style.backgroundColor = captionConrtol_2.style.backgroundColor = 'rgb(0 0 0 / 0.6)';
            captionConrtol_1.style.color = captionConrtol_2.style.color = captionFontColor;
            captionConrtol_1.style.textShadow = captionConrtol_2.style.textShadow = captionFontTextShadow;
            captionConrtol_1.style.webkitTextStroke = captionConrtol_2.style.webkitTextStroke = captionFontTextStroke;
            captionConrtol_1.style.fontWeight = captionConrtol_2.style.fontWeight = '粗体';
            captionConrtol_1.style.display = captionConrtol_2.style.display = '无';
            captionConrtol_1.style.wordBreak = captionConrtol_2.style.wordBreak = '保持所有';

            captionSubWidget.appendChild(captionConrtol_1);
            captionSubWidget.appendChild(captionConrtol_2);
            captionWidget.appendChild(captionSubWidget);

            mainPlayer.prepend(captionWidget);

            captionWidget = document.querySelector('#captionWidget');
            captionSubWidget = document.querySelector('#captionSubWidget');
            captionConrtol_1 = document.querySelector('#captionConrtol_1');
            captionConrtol_2 = document.querySelector('#captionConrtol_2');
        }
    }

    函数包裹()
    {
        返回参数；
    }

    函数 HiddenDownloadSubtitleButton()
    {
        if(hasSubtitleLabel.style.display === 'inline-block')
        {
            hasSubtitleLabel.style.display = 'none';
        }
        if(saveEnSubtitleButton.style.display === 'inline-block')
        {
            saveEnSubtitleButton.style.display = 'none';
        }
        if(saveZhhansSubtitleButton.style.display === 'inline-block')
        {
            saveZhhansSubtitleButton.style.display = 'none';
        }
    }

    函数 HiddenDownloadVideoButton()
    {
        if(downloadVideoList.style.display === 'inline-block')
        {
            downloadVideoList.style.display = 'none';
        }
        if(saveDownloadVideoButton.style.display === 'inline-block')
        {
            saveDownloadVideoButton.style.display = 'none';
        }
    }

    函数 ShowCaption()
    {
        var createElement = document.createElement;
        document.createElement = function(tagName, options)
        {
            var domObject = createElement.apply(document, Wrap(tagName, options));
            if(tagName.toLowerCase() === '视频')
            {
                domObject.addEventListener('结束', function()
                {
                    if(this.classList.contains('html5-main-video'))
                    {
                        HiddenDownloadSubtitleButton();
                        HiddenDownloadVideoButton();
                    }
                });

                domObject.addEventListener('loadstart', function()
                {
                    if(this.classList.contains('html5-main-video'))
                    {
                        
                        if(checkTranslateCaptionTimer !== null)
                        {
                            clearInterval(checkTranslateCaptionTimer);
                            checkTranslateCaptionTimer = null;
                        }
                        checkTranslateCaptionTimer = setInterval(function()
                        {
                            if(HasTranslateCaption())
                            {
                                clearInterval(checkTranslateCaptionTimer);
                                checkTranslateCaptionTimer = null;

                                设置超时（函数（）
                                {
                                    尝试
                                    {
                                        var mainPlayer = domObject;

                                        //创建字幕控件
                                        CreateCaptionControl();
                                        // 字幕下载按钮隐藏，可能没有字幕
                                        HiddenDownloadSubtitleButton();
                                        // 隐藏下载视频按钮
                                        HiddenDownloadVideoButton();

                                        // 修复字幕不切换问题
                                        ResetTranslateCaptionsUrl();
                                        // 获取字幕url
                                        GetTranslateCaptionsUrl();
                                        FetchTranslateCaptionsContent();
                                        
                                        var lastTime = (+new Date());
                                        mainPlayer.addEventListener('timeupdate', function()
                                        {
                                            //优化一下，限制速度30帧，以后进入
                                            var currentTime = (+new Date());
                                            if (currentTime - lastTime > 1000 / 30) {
                                                lastTime = 当前时间；
                                            } 别的 
                                            {
                                                返回;
                                            }
                                            // 获取字幕失败
                                            if(!visiableCaption)
                                            {
                                                返回;
                                            }
                                            // 主动关闭字幕
                                            if(!GetTranslationState()) 
                                            {
                                                返回;
                                            }
                                            //创建字幕控件
                                            CreateCaptionControl();

                                            //最后视频没有字幕，所以导致上个视频字幕，判断按钮隐藏字幕
                                            var subtitlesBtn = document.querySelector('.ytp-subtitles-button');
                                            if(subtitlesBtn === null ||subtitlesBtn.style.display === 'none')
                                            {
                                                返回;
                                            }
    
                                            // 判断模式
                                            var miniPlayer = document.querySelector('.ytp-miniplayer-ui');
                                            if(miniPlayer && miniPlayer.style.display !== 'none')
                                            {
                                                //模拟模式
                                                captionSubWidget.style.bottom = captionFontMiniBottomPosition;
                                                captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontMiniSize;
                                            } 别的 
                                            {
                                                // 正常模式
                                                captionSubWidget.style.bottom = captionFontBottomPosition;
                                                captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontSize;
                                            }
    
    
                                            var time = mainPlayer.currentTime * 1000;
                                            
                                            // 中文字幕
                                            if(captionTranslateContent_1 != null && captionTranslateContent_1 instanceof Array)
                                            {
                                                for(var i=0; i<captionTranslateContent_1.length; i++)
                                                {
                                                    var item = captionTranslateContent_1[i];
                                                    if(!item || !item.segs || !(item.segs instanceof Array)) 
                                                    {
                                                        继续;
                                                    }
                                                    if(time >= item.tStartMs && time <= item.tStartMs + item.dDurationMs)
                                                    {
                                                        var endTime = item.tStartMs + item.dDurationMs - 时间；
                                                        if(captionConrtol_1 !== null)
                                                        {
                                                            尝试
                                                            {
                                                                var text = [];
                                                                for(var k=0; k < item.segs.length; k++) 
                                                                {
                                                                    text.push(item.segs[k].utf8);
                                                                }
                                                                var displayText = text.join(' ');
                                                                displayText = displayText.replace(/\s+/ig, ' ');
                                                                if(captionConrtol_1.innerHTML !== displayText) 
                                                                {
                                                                    captionConrtol_1.innerHTML = displayText;
                                                                }

                                                                if(captionConrtol_1.style.display !== 'block') 
                                                                {
                                                                    captionConrtol_1.style.display = 'block';
                                                                    如果（hiddenCaptionsTimer_1 ！== null）{
                                                                        clearTimeout(hiddenCaptionsTimer_1);
                                                                    }
                                                                    hiddenCaptionsTimer_1 = setTimeout(function()
                                                                    {
                                                                        captionConrtol_1.style.display = '无';
                                                                        clearTimeout(hiddenCaptionsTimer_1);
                                                                        hiddenCaptionsTimer_1 = null;
                                                                    }， 时间结束）;
                                                                }
                                                            }抓住（错误）
                                                            {
                                                                继续;
                                                            }
                                                        }
                                                        休息;
                                                    }
                                                }
                                            }
                                            
                                            // 中文字幕
                                            if(captionTranslateContent_2 != null && captionTranslateContent_2 instanceof Array)
                                            {
                                                for(var i=0; i<captionTranslateContent_2.length; i++)
                                                {
                                                    var item = captionTranslateContent_2[i];
                                                    if(!item || !item.segs || !(item.segs instanceof Array)) 
                                                    {
                                                        继续;
                                                    }
                                                    if(time >= item.tStartMs && time <= item.tStartMs + item.dDurationMs)
                                                    {
                                                        var endTime = item.tStartMs + item.dDurationMs - 时间；
                                                        if(captionConrtol_2 !== null)
                                                        {
                                                            尝试
                                                            {
                                                                var text = [];
                                                                for(var k=0; k < item.segs.length; k++) 
                                                                {
                                                                    text.push(item.segs[k].utf8);
                                                                }
                                                                var displayText = text.join(' ');
                                                                displayText = displayText.replace(/\s+/ig, ' ');
                                                                if(captionConrtol_2.innerHTML !== displayText) 
                                                                {
                                                                    captionConrtol_2.innerHTML = displayText;
                                                                }

                                                                if(captionConrtol_2.style.display !== 'block') 
                                                                {
                                                                    captionConrtol_2.style.display = 'block';
                                                                    if (hiddenCaptionsTimer_2 !== null) {
                                                                        clearTimeout(hiddenCaptionsTimer_2);
                                                                    }
                                                                    hiddenCaptionsTimer_2 = setTimeout(function()
                                                                    {
                                                                        captionConrtol_2.style.display = '无';
                                                                        clearTimeout(hiddenCaptionsTimer_2);
                                                                        hiddenCaptionsTimer_2 = null;
                                                                    }， 时间结束）;
                                                                }
                                                            }抓住（错误）
                                                            {
                                                                继续;
                                                            }
                                                        }
                                                        休息;
                                                    }
                                                }
                                            }
    
                                        });
                                    }抓住（错误）
                                    {

                                    }
                                    
                                }, captionDelayLoadTime);
                            }
                        }, 300);
                        
                    }
                });
            }
            返回 domObject;
        }

    }

    函数 isVideoAdsTime(){
        var ad = document.querySelector('.ad-showing');
        var skipAdButton = document.querySelector('.ytp-ad-skip-button');

        var volumeOpenState = document.querySelector("#ytp-svg-volume-animation-mask");
        var volumeButton = document.querySelector('.ytp-mute-button');

        // 底部广告
        var ads = document.querySelector('.ytp-ad-overlay-container');
        如果（广告！== 空）
        {
            ads.style.display = '无';
        }

        /*
        // 判断有没有广告
        如果（广告）{
            // 关闭全景
            if(volumeOpenState && volumeButton)
            {
                volumeButton.click();
            }
        } 别的 {
            // 正常视频，打开音量
            if(volumeOpenState == null && volumeButton){
                volumeButton.click();
            }
        }
        */
        
        // 跳过广告
        如果（跳过广告按钮）
        {
            skipAdButton.click();
        }
        返回广告 != null;
    }

    函数FuckAds()
    {
        设置间隔（函数（）
        {
            尝试
            {
                isVideoAdsTime();
            }抓住（错误）
            {

            }

        }, 200);
    }

    函数 HookDataUpdate()
    {
        设置间隔（函数（）
        {
            尝试
            {
                var pageManger = document.querySelector('#page-manager');
                如果（页面管理器！== 空）
                {
                    如果（pageManger.isHook ！== 未定义）
                    {
                        返回;
                    }
                    pageManger.isHook = true;
                    var oldUpdatePageData = pageManger.updatePageData;
                    var updatePageData = 函数（数据）
                    {
                        尝试
                        {
                            window.ytInitialPlayerResponse = data.playerResponse;
                            // 修复字幕不切换问题
                            ResetTranslateCaptionsUrl();
                            GetTranslateCaptionsUrl();
                            FetchTranslateCaptionsContent();
                        }抓住（错误）
                        {

                        }
                        
                        返回 oldUpdatePageData.apply(this, arguments);
                    }
                    
                    if(oldUpdatePageData !== updatePageData)
                    {
                       pageManger.updatePageData = updatePageData;
                    }
                }
            }抓住（错误）
            {

            }
        }, 1000);
        
    }

    函数 GetTranslationState()
    {
        return window.localStorage.getItem('double-translation-plugin-state') === 'on';
    }

    函数 AddTranslationButton() 
    {  
        var coltrolPanel = document.querySelector('.ytp-chrome-controls .ytp-right-controls');
        if(coltrolPanel && coltrolPanel.querySelector('.double-translation-plugin-btn') == null) {
            // 开启字幕
            var translationButton = document.createElement('button');
            translationButton.className = 'double-translation-plugin-btn';
            translationButton.style = '位置：相对；顶部：-36%；边距右：10px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';

            translationButton.onclick = function(){
                var translationState = window.localStorage.getItem('double-translation-plugin-state');
                if(translationState == 'on') {
                    window.localStorage.setItem('double-translation-plugin-state', 'off');
                    translationButton.innerText = '开启字幕';
                    // 隐藏字幕
                    captionConrtol_1.style.display = captionConrtol_2.style.display = '无';
                } 别的 {
                    window.localStorage.setItem('double-translation-plugin-state', 'on');
                    translationButton.innerText = '关闭字幕';
                }
            }

            if(GetTranslationState()){
                translationButton.innerText = '关闭字幕';
            } 别的 {
                translationButton.innerText = '开启字幕';
            }
            // 屏蔽youtube强行添加的事件
            translationButton.addEventListener = function() {};
  

            // 添加设置事件

            //字体减大小
            var fontDecSizeButton = document.createElement('button');
            fontDecSizeButton.className = 'double-translation-plugin-font-dec-size-btn';
            fontDecSizeButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            fontDecSizeButton.innerText = '-';
            fontDecSizeButton.onclick = function() {
                var miniPlayer = document.querySelector('.ytp-miniplayer-ui');
                if(miniPlayer && miniPlayer.style.display !== 'none')
                {
                    //模拟模式
                    var fontSize = parseFloat(captionFontMiniSize) - 0.1;
                    如果（字体大小 <= 0.1） 
                    {
                        字体大小 = 0.1
                    }
                    fontSize = fontSize + 'em';
                    captionFontMiniSize = 字体大小；
                    window.localStorage.setItem('double-translation-font-mini-size', captionFontMiniSize);
                    captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontMiniSize;
                } 别的 
                {
                    // 正常模式
                    var fontSize = parseFloat(captionFontSize) - 0.1;
                    如果（字体大小 <= 0.1） 
                    {
                        字体大小 = 0.1
                    }
                    fontSize = fontSize + 'em';
                    标题字体大小 = 字体大小；
                    window.localStorage.setItem('double-translation-font-size', captionFontSize);
                    captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontSize;
                }
            }
            
            //字体加大小
            var fontAddSizeButton = document.createElement('button');
            fontAddSizeButton.className = 'double-translation-plugin-font-add-size-btn';
            fontAddSizeButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            fontAddSizeButton.innerText = '+';
            fontAddSizeButton.onclick = function() {
                var miniPlayer = document.querySelector('.ytp-miniplayer-ui');
                if(miniPlayer && miniPlayer.style.display !== 'none')
                {
                    //模拟模式
                    var fontSize = parseFloat(captionFontMiniSize) + 0.1;
                    如果（字体大小 >= 10） 
                    {
                        字体大小 = 10
                    }
                    fontSize = fontSize + 'em';
                    captionFontMiniSize = 字体大小；
                    window.localStorage.setItem('double-translation-font-mini-size', captionFontMiniSize);
                    captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontMiniSize;
                } 别的 
                {
                    // 正常模式
                    var fontSize = parseFloat(captionFontSize) + 0.1;
                    如果（字体大小 >= 10） 
                    {
                        字体大小 = 10
                    }
                    fontSize = fontSize + 'em';
                    标题字体大小 = 字体大小；
                    window.localStorage.setItem('double-translation-font-size', captionFontSize);
                    captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontSize;
                }
            }
            //字体 上
            var fontUpButton = document.createElement('button');
            fontUpButton.className = 'double-translation-plugin-font-add-size-btn';
            fontUpButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            fontUpButton.innerText = '▲';
            fontUpButton.onclick = function() {
                var miniPlayer = document.querySelector('.ytp-miniplayer-ui');
                if(miniPlayer && miniPlayer.style.display !== 'none')
                {
                    //模拟模式
                    var position = parseFloat(captionFontMiniBottomPosition) + 2;
                    位置 = 位置 + 'px';
                    captionFontMiniBottomPosition = 位置；
                    window.localStorage.setItem('double-translation-font-mini-position', captionFontMiniBottomPosition);
                    captionSubWidget.style.bottom = captionFontMiniBottomPosition;
                } 别的 
                {
                    // 正常模式
                    var position = parseFloat(captionFontBottomPosition) + 2;
                    位置 = 位置 + 'px';
                    captionFontBottomPosition = 位置；
                    window.localStorage.setItem('double-translation-font-position', captionFontBottomPosition);
                    captionSubWidget.style.bottom = captionFontBottomPosition;
                }
            }
            // 字体
            var fontDownButton = document.createElement('button');
            fontDownButton.className = 'double-translation-plugin-font-add-size-btn';
            fontDownButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            fontDownButton.innerText = '▼';
            fontDownButton.onclick = function() {
                var miniPlayer = document.querySelector('.ytp-miniplayer-ui');
                if(miniPlayer && miniPlayer.style.display !== 'none')
                {
                    //模拟模式
                    var position = parseFloat(captionFontMiniBottomPosition) - 2;
                    位置 = 位置 + 'px';
                    captionFontMiniBottomPosition = 位置；
                    window.localStorage.setItem('double-translation-font-mini-position', captionFontMiniBottomPosition);
                    captionSubWidget.style.bottom = captionFontMiniBottomPosition;
                } 别的 
                {
                    // 正常模式
                    var position = parseFloat(captionFontBottomPosition) - 2;
                    位置 = 位置 + 'px';
                    captionFontBottomPosition = 位置；
                    window.localStorage.setItem('double-translation-font-position', captionFontBottomPosition);
                    captionSubWidget.style.bottom = captionFontBottomPosition;
                }
            }
            

            // 变成按钮
            var fontResetButton = document.createElement('button');
            fontResetButton.className = 'double-translation-plugin-font-reset-btn';
            fontResetButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            fontResetButton.innerText = '自我';
            fontResetButton.onclick = function() {

                var miniPlayer = document.querySelector('.ytp-miniplayer-ui');
                if(miniPlayer && miniPlayer.style.display !== 'none')
                {
                    //模拟模式
                    captionFontMiniBottomPosition = '10px';
                    window.localStorage.setItem('double-translation-font-mini-position', captionFontMiniBottomPosition);
                    captionFontMiniSize = '1.4em';
                    window.localStorage.setItem('double-translation-font-mini-size', captionFontMiniSize);

                    captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontMiniSize;
                    captionSubWidget.style.bottom = captionFontMiniBottomPosition;
                } 
                别的 
                {
                    // 正常模式
                    captionFontBottomPosition = '60px';
                    window.localStorage.setItem('double-translation-font-position', captionFontBottomPosition);
                    captionFontSize = '2.0em';
                    window.localStorage.setItem('double-translation-font-size', captionFontSize);

                    captionConrtol_1.style.fontSize = captionConrtol_2.style.fontSize = captionFontSize;
                    captionSubWidget.style.bottom = captionFontBottomPosition;
                }
            }

            // 保存中文字幕
            saveZhhansSubtitleButton = document.createElement('a');
            saveZhhansSubtitleButton.style.display = 'none';
            var _saveZhhansSubtitleButton = document.createElement('button');
            _saveZhhansSubtitleButton.className = 'double-translation-plugin-download-zhhans-subtitle-btn';
            _saveZhhansSubtitleButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            _saveZhhansSubtitleButton.innerText = '中字';
            saveZhhansSubtitleButton.appendChild(_saveZhhansSubtitleButton);

            // 保存中文字幕
            saveEnSubtitleButton = document.createElement('a');
            saveEnSubtitleButton.style.display = 'none';
            var _saveEnSubtitleButton = document.createElement('button');
            _saveEnSubtitleButton.className = 'double-translation-plugin-download-en-subtitle-btn';
            _saveEnSubtitleButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            _saveEnSubtitleButton.innerText = '英字';
            saveEnSubtitleButton.appendChild(_saveEnSubtitleButton);


            hasSubtitleLabel = document.createElement('span');
            hasSubtitleLabel.className = 'double-translation-plugin-download-label';
            hasSubtitleLabel.style = '位置：相对；顶部：-36%；边距右：3px; 颜色：#fff; 大纲：无；字体粗细：粗体；显示：无；';
            hasSubtitleLabel.innerText = '下载字幕：';


            hasFetchTranslationButton = document.createElement('button');
            hasFetchTranslationButton.className = 'double-translation-plugin-has-translation-btn';
            hasFetchTranslationButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            hasFetchTranslationButton.innerText = '正常';

            // 视频下载
            downloadVideoList = document.createElement('select');
            downloadVideoList.className = 'double-translation-plugin-download-video-list';
            downloadVideoList.style = '位置：相对；顶部：-36%；边距右：3px; 不透明度：0.95；显示：无;';
            downloadVideoList.onchange = function()
            {
                if(this.value === '无')
                {
                    返回;
                } 别的 
                {
                    // window.open(this.value, '_new' , '视频下载');
                    var fileContent = this.value;
                    var 文件名 = window.ytInitialPlayerResponse.videoDetails.title + '.mp4';
                    saveDownloadVideoButton.setAttribute('download-url', fileContent);
                    saveDownloadVideoButton.setAttribute('下载', 文件名);
                }
            }

            saveDownloadVideoButton = document.createElement('a');
            saveDownloadVideoButton.style.display = 'none';
            saveDownloadVideoButton.href = 'javascript:;';

            var _saveDownloadVideoButton = document.createElement('button');
            _saveDownloadVideoButton.className = 'double-translation-plugin-download-en-subtitle-btn';
            _saveDownloadVideoButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无；';
            _saveDownloadVideoButton.innerText = '下载';
            _saveDownloadVideoButton.onclick = function()
            {
                var downUrl = this.parentElement.getAttribute('download-url');
                如果（！downUrl） 
                {
                    alert('请先选择需要下载的视频');
                } 别的 
                {
                    prompt('请复制地址到迅雷进行下载', downUrl);
                }
                
            }
            saveDownloadVideoButton.appendChild(_saveDownloadVideoButton);
            
            //刷新视频下载地址
            var refreshVideoButton = document.createElement('a');
            refreshVideoButton.href = 'javascript:;';
            var _refreshVideoButton = document.createElement('button');
            _refreshVideoButton.className = 'double-translation-plugin-download-en-subtitle-btn';
            _refreshVideoButton.style = '位置：相对；顶部：-36%；边距右：3px; 边界半径：25px；边界：无；不透明度：0.95；背景色：#fff; 大纲：无;';
            _refreshVideoButton.innerText = '刷新';
            _refreshVideoButton.onclick = function()
            {
                HiddenDownloadVideoButton();
                GetCurrentVideoPlayerUrls();
            }
            refreshVideoButton.appendChild(_refreshVideoButton);


            // 管理面板
            var panel = document.createElement('div');
            panel.style.position = '绝对';
            panel.style.bottom = '100%';
            panel.style.right = '0';


            panel.prepend(translationButton);
            panel.prepend(hasFetchTranslationButton);
            panel.prepend(fontResetButton);
            panel.prepend(fontAddSizeButton);
            panel.prepend(fontDecSizeButton);
            panel.prepend(fontDownButton);
            panel.prepend(fontUpButton);
            
            panel.prepend(saveZhhansSubtitleButton);
            panel.prepend(saveEnSubtitleButton);
            // panel.prepend(hasSubtitleLabel);

            panel.prepend(saveDownloadVideoButton);
            panel.prepend(downloadVideoList);

            panel.prepend(refreshVideoButton);
            coltrolPanel.prepend(panel);
            返回真；
        }
    }

    函数CalcTime（时间）
    {
        var second = Math.floor(time / 1000);
        var 分钟 = Math.floor(second / 60);
        var 小时 = Math.floor(minute / 60);
        分钟 = Math.floor(分钟 - 小时 * 60);
        second = Math.floor(second - minute * 60 - hour * 60 * 60);
        ms = Math.floor(time - (second + minute * 60 + hour * 60 * 60));

        小时 = '00' + 小时
        分钟 = '00' + 分钟
        秒 = '00' + 秒
        毫秒 = 毫秒 + '000'

        小时 = 小时.substr(小时.长度 - 2, 小时.长度);
        分钟 = 分钟.substr(分钟.长度 - 2, 分钟.长度);
        second = second.substr(second.length - 2, second.length);
        ms = ms.substr(0, 3);
        返回 [小时，分钟，秒].join(':') + '.' + 毫秒；
    }

    // 生成中文字幕
    函数 GenerateSRTFromZhhans() {
        var 结果 = [];
        无功指数 = 0;
        if(captionTranslateContent_1 != null && captionTranslateContent_1 instanceof Array)
        {
            for(var i=0; i<captionTranslateContent_1.length; i++)
            {
                var item = captionTranslateContent_1[i];
                if(!item || !item.segs || !(item.segs instanceof Array)) 
                {
                    继续;
                }
                var text = [];
                for(var k=0; k < item.segs.length; k++) 
                {
                    text.push(item.segs[k].utf8);
                }
                var displayText = text.join('').trim();
                displayText = displayText.replace(/\s+/ig, ' ');
                var displayTime = [CalcTime(parseInt(item.tStartMs)), CalcTime(parseInt(item.tStartMs) + parseInt(item.dDurationMs))].join(' --> ');
                if(displayText.length > 0) 
                {
                    result.push(++index);
                    结果。推（显示时间）；
                    result.push(displayText + '\n');
                }
            }
        }

        return result.join('\n');
    }

    // 生成中文字幕
    函数 GenerateSRTFromEn()
    {
        var 结果 = [];
        无功指数 = 0;
        if(captionTranslateContent_2 != null && captionTranslateContent_2 instanceof Array)
        {
            for(var i=0; i<captionTranslateContent_2.length; i++)
            {
                var item = captionTranslateContent_2[i];
                if(!item || !item.segs || !(item.segs instanceof Array)) 
                {
                    继续;
                }
                var text = [];
                for(var k=0; k < item.segs.length; k++) 
                {
                    text.push(item.segs[k].utf8);
                }
                var displayText = text.join(' ').trim();
                displayText = displayText.replace(/\s+/ig, ' ');
                var displayTime = [CalcTime(parseInt(item.tStartMs)), CalcTime(parseInt(item.tStartMs) + parseInt(item.dDurationMs))].join(' --> ');
                if(displayText.length > 0) 
                {
                    result.push(++index);
                    结果。推（显示时间）；
                    result.push(displayText + '\n');
                }
            }
        }
        return result.join('\n');
    }

    函数 DecodeURIParamToDict(a) 
    {
        a = a.split("&");
        for (var b = {}, c = 0, d = a.length; c < d; c++) {
            var e = a[c].split("=");
            if (1 == e.length && e[0] || 2 == e.length)
                尝试 {
                    var f = decodeURIComponent(e[0] || "")
                      , h = decodeURIComponent(e[1] || "");
                    b 中的 f ? Array.isArray(b[f]) ? vb(b[f], h) : b[f] = [b[f], h] : b[f] = h
                }赶上（米）{
                    如果（“q”！= e[0]）{
                        var l = Error("URL 组件解码错误");
                        l.params = {
                            键：e[0]，
                            值：e[1]
                        };
                        投掷（升）
                    }
                }
        }
        返回 b
    };

    // youtube视频
    函数 DecryptBySignatureCipher(s)
    {
        无功 = {
            oG：函数（a）{
                a.反向（）
            },
            欧盟：函数（a，b）{
                var c = a[0];
                a[0] = a[b % a.length];
                a[b % a.length] = c
            },
            zH：函数（a，b）{
                a.splice(0, b)
            }
        };  
        
        a = s.split("");
        Vu.Eu(a, 29);
        Vu.oG(a, 9);
        Vu.Eu(a, 38);
        Vu.oG(a, 66);
        返回 a.join("")
    }

    函数主（）
    {
        他妈的广告（）；
        // 不能挂ajax，所以只能这样更新更新数据
        setInterval(AddTranslationButton, 500);
        钩数据更新();
        ShowCaption();
    }
    

    主要的（）;

})();
