---
title: opengl中绘制文字办法
date: 2021-06-30 14:59:37
tags:
---


使用freetype库可以方便的加载字体库中的字符，将字符图片上传成纹理，再将纹理绘制出来。

## 使用文档
https://learnopengl-cn.readthedocs.io/zh/latest/06%20In%20Practice/02%20Text%20Rendering/

https://learnopengl.com/code_viewer.php?code=in-practice/text_rendering

## 效果
![](https://mweb-image-1259394369.cos.ap-guangzhou.myqcloud.com/2021/06/30/16250368407535.jpg)


## 我的实现

TextProgram.h
```
//
//  TextProgram.hpp
//  kiwi
//
//  Created by justinsong on 2021/6/28.
//


#pragma once

#include "../interface.h"
#include <ai/face/Type.h>
#include <list>


struct TextInfo {
    std::string text;
    kiwi::fvec2 coodr;
    kiwi::fvec3 color;
    float scale;
};


class TextProgram : public GLProgram {
public:
    void init() override;
    
    void init(const char * fontPath);
    
    void setTextInfo(std::list<TextInfo> textList);
    
protected:
    
    int onDraw(GLParams &glParams) override;
    
private:
    
    void initFT(const char * fontPath);
    void RenderText(std::string text, GLfloat x, GLfloat y, GLfloat scale, kiwi::fvec3 color);

    
    GLfloat _width = 0.0;
    GLfloat _height = 0.0;
    std::list<TextInfo> _textList;
};


```


TextProgram.cpp

```
//
//  TextProgram.cpp
//  kiwi
//
//  Created by justinsong on 2021/6/28.
//

#include "TextProgram.h"
#include <iostream>
#include <map>
#include <string>
// FreeType
#include <ft2build.h>
#include FT_FREETYPE_H
#include "IKiwiService.h"

struct Character {
    GLuint TextureID;   // ID handle of the glyph texture
    kiwi::fvec2 Size;    // Size of glyph
    kiwi::fvec2 Bearing;  // Offset from baseline to left/top of glyph
    GLuint Advance;    // Horizontal offset to advance to next glyph
};

std::map<GLchar, Character> Characters;

void TextProgram::init() {
    
}

void TextProgram::init(const char *fontPath) {
    mProgramHandle = util::gl::createProgram(kiwi::getShader("shaders/text/text.vs"), kiwi::getShader("shaders/text/text.frag"));
    initFT(fontPath);
}

void TextProgram::initFT(const char *fontPath) {
    // FreeType
    FT_Library ft;
    // All functions return a value different than 0 whenever an error occurred
    if (FT_Init_FreeType(&ft))
        std::cout << "ERROR::FREETYPE: Could not init FreeType Library" << std::endl;
    
    // Load font as face
    FT_Face face;
    
    if (FT_New_Face(ft, fontPath, 0, &face))
        std::cout << "ERROR::FREETYPE: Failed to load font" << std::endl;
    
    // Set size to load glyphs as
    FT_Set_Pixel_Sizes(face, 0, 48);
    
    // Disable byte-alignment restriction
    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
    
    // Load first 128 characters of ASCII set
    for (GLubyte c = 0; c < 128; c++)
    {
        // Load character glyph
        if (FT_Load_Char(face, c, FT_LOAD_RENDER))
        {
            std::cout << "ERROR::FREETYTPE: Failed to load Glyph" << std::endl;
            continue;
        }
        // Generate texture
        GLuint texture;
        glGenTextures(1, &texture);
        glBindTexture(GL_TEXTURE_2D, texture);
        glTexImage2D(
                     GL_TEXTURE_2D,
                     0,
                     GL_RED,
                     face->glyph->bitmap.width,
                     face->glyph->bitmap.rows,
                     0,
                     GL_RED,
                     GL_UNSIGNED_BYTE,
                     face->glyph->bitmap.buffer
                     );
        
        // Set texture options
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        // Now store character for later use
        Character character;
        character.TextureID = texture;
        character.Size = kiwi::fvec2(face->glyph->bitmap.width, face->glyph->bitmap.rows);
        character.Bearing = kiwi::fvec2(face->glyph->bitmap_left, face->glyph->bitmap_top);
        character.Advance = (GLuint)face->glyph->advance.x;

        Characters.insert(std::pair<GLchar, Character>(c, character));
    }
    glBindTexture(GL_TEXTURE_2D, 0);
    // Destroy FreeType once we're finished
    FT_Done_Face(face);
    FT_Done_FreeType(ft);
}

void TextProgram::RenderText(std::string text, GLfloat x, GLfloat y, GLfloat scale, kiwi::fvec3 color) {
    
    // Iterate through all characters
    std::string::const_iterator c;
    
    for (c = text.begin(); c != text.end(); c++)
    {
        Character ch = Characters[*c];
        
        GLfloat xpos = x + ch.Bearing.x * scale;
        GLfloat ypos = y - (ch.Size.y - ch.Bearing.y) * scale;
        
        GLfloat w = ch.Size.x * scale;
        GLfloat h = ch.Size.y * scale;
        
        GLfloat x_w = xpos + w;
        GLfloat y_h = ypos + h;
        GLfloat xCoodr = xpos;
        GLfloat yCoodr = ypos;
        
        x_w = (x_w / _width) * 2.0 - 1;
        y_h = (y_h / _height) * 2.0 - 1;
        xCoodr = (xCoodr / _width) * 2.0 - 1;
        yCoodr = (yCoodr / _height) * 2.0 - 1;
        
        GLfloat vertxCoodr[] = {
            x_w, y_h,
            x_w, yCoodr,
            xCoodr, y_h,
            xCoodr, yCoodr
        };
        
        GLfloat texCoodr[] = {
            1.0, 1.0,
            1.0, 0.0,
            0.0, 1.0,
            0.0, 0.0
        };
        
        GLfloat clr[] = {color.x, color.y, color.z};
        
        // Render glyph texture over quad
        bindTexture("uTexture", ch.TextureID, 1);
        bindCoordinate("aPosition", FULL_COORDS_PER_VERTEX, vertxCoodr);
        bindCoordinate("aTexCoordinate", FULL_COORDS_PER_VERTEX, texCoodr);
        setUniform3fv("textColor", 1, clr);
        
        glDrawArrays(GL_TRIANGLE_STRIP, 0, FULL_VERTEX_COUNT);
        
        // Now advance cursors for next glyph (note that advance is number of 1/64 pixels)
        x += (ch.Advance >> 6) * scale; // Bitshift by 6 to get value in pixels (2^6 = 64 (divide amount of 1/64th pixels by 64 to get amount of pixels))
        GLASSERT();
    }
    
    GLASSERT();
}


int TextProgram::onDraw(GLParams &glParams) {
    glUseProgram(mProgramHandle);
    
    _width = glParams.viewport.width;
    _height = glParams.viewport.height;
    
    std::list<TextInfo>::iterator it; //声明一个迭代器
    for(it = _textList.begin(); it != _textList.end(); it++){
        TextInfo textInfo = *it;
        this->RenderText(textInfo.text, textInfo.coodr.x, textInfo.coodr.y, textInfo.scale, textInfo.color);
    }
    
    GLASSERT();
    return 0;
}

void TextProgram::setTextInfo(std::list<TextInfo> textList) {
    _textList = textList;
}

```