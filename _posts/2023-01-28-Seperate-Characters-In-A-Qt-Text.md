---
layout:     post
title:      Seperate Characters In A Qt Text
subtitle:   QTextDocument
date:       2023-01-28
author:     Yves
header-img: "img/post-bg.jpg"
catalog: true
tags:
    - C++ 
    - Qt
---

## Describe
Implement a function that can **seperate a Qt text** from one document(entity) into multiple documents(entities), with the key focus being on determining the distance between adjacent characters (the intervals in below graph).

## Effect
Before Seperating, 'Hello World' is a entity.
![image](/img/20230128/6.1.png)

After Seperating, every characters are entities.
![image](/img/20230128/6.2.png)

## Structure
Each entity holds a [QTextDocument](https://doc.qt.io/qt-6/qtextdocument.html) class composed of more than one [QTextBlock](https://doc.qt.io/qt-6/qtextblock.html) class, where each 'Block' represents a single line.  
![image](/img/20230128/6.3.png)

Key variables are the blockHeight and every intervals.
![image](/img/20230128/6.4.png)

## Code
```

void seperate(){
    // count the total block height
    float blockHeight = 0;
    // get format from qt document
    auto format = QTextCursor(docoument).charFormat();


    // block loop
    for (int j = 0; j < docoument->blockCount(); j++) 
    {       
        QTextBlock block = docoument->findBlockByNumber(j); // get current block
        QTextCursor textCursor(block); // get current cursor 
        QString context = block.text(); // get current context
        QTextLine line = block.layout()->lineAt(0); //get current line
        
        QList<QGlyphRun> glyphs;
        QVector<QVector<QPointF>> pos; // save each word position
        
        // loop of evey word in the block
        for (int i = 0; i < context.length(); i++)
        {
            glyphs.append(line.glyphRuns(i, 1));
            // get the interval from fisrt word to current word (img 3)
            auto interval = glyphs[i].positions(); 
            pos.push_back(interval); 

            auto new_item = new TextDocumentItem();
            QTextCursor new_cursor(new_item->get_document());

            // set new format 
            new_cursor.setBlockCharFormat(new_cursor.blockCharFormat());
            new_cursor.setCharFormat(textCursor.charFormat());

            // set text context
            new_cursor.insertText(QString{ context[i] });

            // interval(pos[i][0].x(), blockHeight)
            // new document position(orignal.x + pos[i][0].x(), orignal.y - blockHeight)

        }
        blockHeight += line.height();
    }
}
```