# libyuv
https://github.com/lemenkov/libyuv/blob/master/docs/getting_started.md

## 注意大小端
https://github.com/lemenkov/libyuv/blob/master/docs/formats.md#the-argb-fourcc

## build for ios
https://github.com/Jichao/libyuv-ios/blob/master/build.sh

## cocos v1.9 使用 xcode 在 ios simulator 构建时需要排除 libyuv_neon.a 这个ARM库（新版本的 cocos 走 cmake 构建，避免了这个问题）
Build Options->Excluded Source File Names->Debug->Any iOS Simulator SDK: libyuv_neon.a

# ijkplayer

## 解析 IJKPlayer
http://www.samirchen.com/ijkplayer/

## ijkplayer 仿SDL
SDL 1.2 不支持 Android/iOS，而原生 ffplay 用到的接口在 SDL 2.0 里没有。
ijkplayer 参考 SDL 2.0 的接口，实现了仿SDL的 ijksdl https://github.com/bilibili/ijkplayer/issues/56#issuecomment-50605031
https://zhuanlan.zhihu.com/p/45237178

## xcode error can't locate file for: -lavfilter; file: -lavfilter is not an object file (not allowed in a library)
因为没有配Library Search Paths

## ijkmp_start()=-3
起初msg_loop里没有实现ijkmp_get_msg时，调用ijkmp_start出现此错误

# HDLPlayer 将 ijkplayer 集成进 cocos

创建 _mediaPlayer 时指定我们自己的 SDL_Vout，实现它的 display_overlay 回调函数，参数 overlay 就是帧解码后的数据。 
```c
static int vout_display_overlay_l(SDL_Vout *vout, SDL_VoutOverlay *overlay) {
    SDL_Vout_Opaque *opaque = vout->opaque;
    opaque->overlay         = overlay;
    return 0;
}

SDL_Vout *SDL_VoutCC_CreateForImageInfo() {
    SDL_Vout *vout = SDL_Vout_CreateInternal(sizeof(SDL_Vout_Opaque));
    if (!vout)
        return NULL;

    SDL_Vout_Opaque *opaque = vout->opaque;
    opaque->overlay         = nullptr;
    vout->create_overlay    = vout_create_overlay;
    vout->free_l            = vout_free_l;
    vout->display_overlay   = vout_display_overlay;

    return vout;
}

IjkMediaPlayer *ijkmp_cc_create(int (*msg_loop)(void*))
{
    IjkMediaPlayer *mp = ijkmp_create(msg_loop);
    if (!mp)
        goto fail;

    mp->ffplayer->vout = SDL_VoutCC_CreateForImageInfo();
    if (!mp->ffplayer->vout)
        goto fail;

    mp->ffplayer->pipeline = ffpipeline_create_from_cc(mp->ffplayer);
    if (!mp->ffplayer->pipeline)
        goto fail;

    return mp;

fail:
    ijkmp_dec_ref_p(&mp);
    return NULL;
}

_mediaPlayer = ijkmp_cc_create(media_player_msg_loop);
```

设置 overlay-format 为 fcc-rv32， ijkplayer 会为我们做数据格式转换，此时 overlay->pixels[0] 是转换后的 ABGR（我们得到的实际上是RGBA） 数据。
```c
ijkmp_set_option(_mediaPlayer, IJKMP_OPT_CATEGORY_PLAYER, "overlay-format", "fcc-rv32")

static int func_fill_frame(SDL_VoutOverlay *overlay, const AVFrame *frame)
{
    ...
    switch (overlay->format) {
        ...
        case SDL_FCC_RV32:
            dst_format = AV_PIX_FMT_0BGR32;
            break;
        case SDL_FCC_RV24:
            dst_format = AV_PIX_FMT_RGB24;
            break;
        case SDL_FCC_RV16:
            dst_format = AV_PIX_FMT_RGB565;
            break;
        default:
            ALOGE("SDL_VoutFFmpeg_ConvertPicture: unexpected overlay format %s(%d)",
                  (char*)&overlay->format, overlay->format);
            return -1;
    }

    ...

    if (use_linked_frame) {
        // do nothing
    } else if (ijk_image_convert(frame->width, frame->height,
                                 dst_format, swscale_dst_pic.data, swscale_dst_pic.linesize,
                                 frame->format, (const uint8_t**) frame->data, frame->linesize)) {
        opaque->img_convert_ctx = sws_getCachedContext(opaque->img_convert_ctx,
                                                       frame->width, frame->height, frame->format, frame->width, frame->height,
                                                       dst_format, opaque->sws_flags, NULL, NULL, NULL);
        if (opaque->img_convert_ctx == NULL) {
            ALOGE("sws_getCachedContext failed");
            return -1;
        }

        sws_scale(opaque->img_convert_ctx, (const uint8_t**) frame->data, frame->linesize,
                  0, frame->height, swscale_dst_pic.data, swscale_dst_pic.linesize);

        if (!opaque->no_neon_warned) {
            opaque->no_neon_warned = 1;
            ALOGE("non-neon image convert %s -> %s", av_get_pix_fmt_name(frame->format), av_get_pix_fmt_name(dst_format));
        }
    }
    
    // TODO: 9 draw black if overlay is larger than screen
    return 0;
}
```

拿到 RBGA 数据后，仿照 HTMLImageElement 的 jsb.loadImage
```js
class HTMLImageElement extends HTMLElement {
    set src(src) {
        this._src = src;
        if (src === '') return;
        jsb.loadImage(src, (info) => {
            if (!info) {
                this._data = null;
                var event = new Event('error');
                this.dispatchEvent(event);
                return;
            }
            this.width = this.naturalWidth = info.width;
            this.height = this.naturalHeight = info.height;
            this._data = info.data;
            this.complete = true;

            var event = new Event('load');
            this.dispatchEvent(event);
        });
    }
}
```
我们也返回同样的数据给 JS
```cpp
static bool HDLPlayer_takeImageInfo(se::State& s) {
    HDLPlayer* cobj = (HDLPlayer*)s.nativeThisObject();
    SE_PRECONDITION2(cobj, false, "HDLPlayer_takeImageInfo : Invalid Native Object");

    auto photo = cobj->takeImageInfo();
    if(!photo) {
        s.rval().setNull();
        return true;
    }

    se::Value dataVal;
    dataVal.setUint64(reinterpret_cast<uintptr_t>(photo->data));

    se::HandleObject retObj(se::Object::createPlainObject());
    retObj->setProperty("data", dataVal);
    retObj->setProperty("width", se::Value(photo->width));
    retObj->setProperty("height", se::Value(photo->height));

    s.rval().setObject(retObj);

    return true;
}
SE_BIND_FUNC(HDLPlayer_takeImageInfo)
```
最后我们用Image（它是HTMLImageElement的派生类）接收数据
```ts
let prototype = (jsb.HDLPlayer as Function).prototype;

prototype.getTexImageSource = function(): TexImageSource | null {
    const info = this.takeImageInfo();
    if (!info) return null;

    if (!this._image) this._image = new Image()

    this._image.width = this._image.naturalWidth = info.width;
    this._image.height = this._image.naturalHeight = info.height;
    this._image._data = info.data;
    
    return this._image;
}

const image = this._player.getTexImageSource();
if (!image) return;
const texture = new Texture2D();
texture.reset({width: image.width, height: image.height})
texture.uploadData(image as any);
const spriteFrame = new SpriteFrame();
spriteFrame.texture = texture;
this.sprite.spriteFrame = spriteFrame;
```

# web 端实现
```ts
export class HDLPlayer {
    private _video: HTMLVideoElement = document.createElement('video');

    init (url: string): void {
        const source = document.createElement('source');
        source.src = url;
        this._video.appendChild(source);
        this._video.load();
    }

    play () {
        const promise = this._video.play();
    }

    getTexImageSource ():TexImageSource | null {
        this._video.width = this._video.videoWidth;
        this._video.height = this._video.videoHeight;
        return this._video;
    }
}
```

# 其它参考资料
## 基于 FFmpeg 的 Cocos Creator 视频播放器
https://oedx.github.io/2020/12/29/FFmpeg-Cocos-Creator/

