---
title: SDL say helloword with matrix format png
date: 2016-11-27 16:04:32
tags:ndk android studio sdl
---
＃# 1.android studio ndk开发配置
用sdk mannerger下载好ndk,设置项目支持ndk开发,项目中build.gradle 配置如下：
' sourceSets.main{
        jniLibs.srcDir new File(projeclibs')
//        jni.srcDirs=[]
    }
    task buildNative(type: Exec, description: 'Compile JNI source via NDK') {
        def ndkDir = android.ndkDirectory
        commandLine "$ndkDir/ndk-build",
                '-C', file('jni').absolutePath, // Change src/main/jni the relative path to your jni source
                '-j', Runtime.runtime.availableProcessors(),
                'all',
                'NDK_DEBUG=1'
    }

    task cleanNative(type: Exec, description: 'Clean JNI object files') {
        def ndkDir = android.ndkDirectory
        commandLine "$ndkDir/ndk-build",
                '-C', file('jni').absolutePath, // Change src/main/jni the relative path to your jni source
                'clean'
    }

    clean.dependsOn 'cleanNative'

    tasks.withType(JavaCompile) {
        compileTask -> compileTask.dependsOn buildNative
    }'

    ## 2.sdl main函数编写
    'struct SDL_Window *window = NULL;
    struct SDL_Renderer *render = NULL;
    struct SDL_Surface *bmp = NULL;
    struct SDL_Texture *texture = NULL;

    int mWidth;
    		int mHeight;

    		void* mPixels;
            		int mPitch;

    char *filepath = "portrait.png";

    if (SDL_Init(SDL_INIT_VIDEO | SDL_INIT_AUDIO | SDL_INIT_TIMER) == -1) {
        LOGE("SDL_Init failed %s", SDL_GetError());
    }

    window = SDL_CreateWindow("SDL HelloWorld!", 100, 100, 640, 480,
            SDL_WINDOW_SHOWN);
    if (window == NULL) {
        LOGE("SDL_CreateWindow failed  %s", SDL_GetError());
    }

    render = SDL_CreateRenderer(window, -1,
            SDL_RENDERER_ACCELERATED | SDL_RENDERER_PRESENTVSYNC);
    if (render == NULL) {
       LOGE("SDL_CreateRenderer failed  %s", SDL_GetError());
    }
    bmp = IMG_Load(filepath);
    LOGE("go image_load fail", SDL_GetError());
    if (bmp == NULL) {
            LOGE("SDL_LoadBMP failed: %s", SDL_GetError());
        }
    SDL_Surface* formattedSurface = SDL_ConvertSurfaceFormat(bmp, SDL_PIXELFORMAT_RGBA8888, NULL );
    if( formattedSurface == NULL )
     		{
     			SDL_Log( "Unable to convert loaded surface to display format! %s\n", SDL_GetError() );
     		}
     		else
     		{
     		texture =SDL_CreateTexture(render, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_STREAMING, formattedSurface->w, formattedSurface->h );
     		LOGE("succuss create texture wait to display", SDL_GetError());
     		if( texture == NULL )
            			{
            				SDL_Log( "Unable to create blank texture! SDL Error: %s\n", SDL_GetError() );
            			}
            			else
            			{
            				//Enable blending on texture
            				//Enable blending on texture
                            				SDL_SetTextureBlendMode( texture, SDL_BLENDMODE_BLEND );

                            				//Lock texture for manipulation
                            				SDL_LockTexture( texture, &formattedSurface->clip_rect, &mPixels, &mPitch );

                            				//Copy loaded/formatted surface pixels
                            				memcpy( mPixels, formattedSurface->pixels, formattedSurface->pitch * formattedSurface->h );

                            				//Get image dimensions
                            				mWidth = formattedSurface->w;
                            				mHeight = formattedSurface->h;

                            				//Get pixel data in editable format
                            				Uint32* pixels = (Uint32*)mPixels;
                            				int pixelCount = ( mPitch / 4 ) * mHeight;

                            				//Map colors
                            				Uint32 colorKey = SDL_MapRGB( formattedSurface->format, 0, 0xFF, 0xFF );
                            				Uint32 transparent = SDL_MapRGBA( formattedSurface->format, 0x00, 0xFF, 0xFF, 0x00 );

                            				//Color key pixels
                            				for( int i = 0; i < pixelCount; ++i )
                            				{
                            					if( pixels[ i ] == colorKey )
                            					{
                            						pixels[ i ] = transparent;
                            					}
                            				}

                            				//Unlock texture to update
                            				SDL_UnlockTexture( texture );
                            				mPixels = NULL;}
     		}


   // texture = SDL_CreateTextureFromSurface(render, bmp);
    SDL_FreeSurface(bmp);
    SDL_FreeSurface(formattedSurface);

    SDL_RenderClear(render);
    SDL_RenderCopy(render, texture, NULL, NULL);
    SDL_RenderPresent(render);

    SDL_Delay(10000);'

    注意图片的路径，sdl会以3中形式寻找图片1、绝对路径 2.相对路径 3.asset
    显示的流程很简单，render 显示在哪 texture显示的内容，代码中大部分都在制造显示的内容，支持多种格式的图片。


    ＃＃ 3.github项目源码地址
    [项目源码地址](https://github.com/highway33/vedio.git)