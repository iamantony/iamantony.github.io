---
title: qtcsv: working with csv-files in Qt
layout: default
author:
  name: Antony Cherepanov
  url: {{ site.url }}
---

About a year ago, I worked on my home-project that has relation to image
compression. I used my favorite framework - Qt - and things were going
well.

Until I didn't have to plot some statistics data. If I were used Python,
I would just import [matplotlib][1] library, add several lines of code
and - whoop - get neat diagrams! However I used C++ and Qt. In addition, I
didn't want to make this task too complicated. Therefore, I decided that the
simplest way for me was to export my data to [csv-file][2], open it with
Microsoft Excel or LibreOffice Calc and create plots.

I don't intend to describe here the last part of this task - creation of
plots in the office suite. There are bunch of good articles and videos
on this topic [in the Internet][3]. Instead I want to concentrate on the
csv-stuff: how to read and write csv files using Qt.

Ok, let's define our task in more detail. For example, we have a list of
[pixel][4] objects and we want to create a [histograms][5] for each
component of those pixels. Each pixel have three channels: red, green and
blue. So this data could be represented as this table:

| red | green | blue |
| --- | ----- | ---- |
|  11 |   255 |    0 |
| 167 |   209 |   89 |
|  34 |    55 |   12 |
| ... | ..... | .... |

How we can export such table to csv-file? First, let's see what is
a csv-file. csv-file is a text file with quite a simple structure. Each
line in csv-file is file interpreted as a single row of a table. Elements
(or cell values) in the each row are separated with separator symbol (comma
or tab or any other separator, even "avb!@;;"). So our table in csv-file
will looks like this (with comma as separator symbol):

``` text
red,green,blue
11,255,0
167,209,89
34,55,12
...
```

Good. Next question - how to create such file programmatically? No
problem, all the necessary code you'll find in this example project:

https://github.com/iamantony/csv-read-write-example

Here is the code from main.cpp file:

``` cpp
#include <QList>
#include <QColor>
#include <QString>
#include <QStringList>
#include <QFile>
#include <QTextStream>
#include <QDebug>

QList<QRgb> GetPixels()
{
    QList<QRgb> pixels;
    pixels << qRgb(11, 255, 0) << qRgb(167, 209, 89) << qRgb(34, 55, 12);

    return pixels;
}

QList<QStringList> PixelsToStrings(const QList<QRgb>& pixels)
{
    QList<QStringList> strings;
    for (int i = 0; i < pixels.size(); ++i)
    {
        QStringList values;
        values << QString::number(qRed(pixels.at(i))) <<
                  QString::number(qGreen(pixels.at(i))) <<
                  QString::number(qBlue(pixels.at(i)));

        strings << values;
    }

    return strings;

}

void WriteToCSV(const QList<QStringList>& pixels)
{
    // Open csv-file
    QFile file("pixels.csv");
    file.open(QIODevice::Append | QIODevice::Text);

    // Write data to file
    QTextStream stream(&file);
    QString separator(",");
    for (int i = 0; i < pixels.size(); ++i)
    {
        stream << pixels.at(i).join(separator) << endl;
    }

    stream.flush();
    file.close();
}

QList<QStringList> ReadCSV()
{
    // Open csv-file
    QFile file("pixels.csv");
    file.open(QIODevice::ReadOnly | QIODevice::Text);

    // Read data from file
    QTextStream stream(&file);
    QList<QStringList> data;
    QString separator(",");
    while (stream.atEnd() == false)
    {
        QString line = stream.readLine();
        data << line.split(separator);
    }

    file.close();
    return data;
}

void Print(const QList<QStringList>& data)
{
    for (int i = 0; i < data.size(); ++i)
    {
        qDebug() << data.at(i).join(", ");
    }
}

int main()
{
    QList<QRgb> pixels = GetPixels();
    QList<QStringList> pixelsStr = PixelsToStrings(pixels);
    WriteToCSV(pixelsStr);

    QList<QStringList> readData = ReadCSV();
    Print(readData);

    return 0;
}
```

Code, that create and write csv-file is located in function *WriteToCSV()*.
It's very short, as you see. If you compile this project and run it,
in project folder (or in build folder) will be created file with name
*pixels.csv*. You can open it with your favorite text editor, you will
see the desired numbers.

Reading of the csv-file isn't a difficult task (also). In function
*ReadCSV()* we open csv-file, read it line by line and split these
lines by separator symbol. As the output, we get list of lists of
strings - elements of the table.

You can use csv-files as a method to *export* data from your program
and as a method to *import* data into your program. Very useful!

Code that I posted here for reading and writing csv-files is Ok.
But it have some drawbacks.

1. Use only one type of data - strings. I mean, functions use list of
strings as output/input objects, so before using these functions
you must transform your data (it could be integers, floats or other
complex objects) to strings by yourself.

2. High memory consumption.
    
  2.1. When you read the csv-file, all it's content will be loaded
  to the memory. It's Ok when file is small, but what if it has
  hundreds of thousands of lines? It's highly possible that
  eventually you'll run out of memory and your program will crush.
  
  2.2. When you write to the csv-file all your data primarily have to
  be converted to strings. It is OK if your data contains only strings,
  so no conversion is needed. Otherwise, in memory you'll have two identical
  copies of your data (original and stringified). After that when you stream
  your data to the file it will be at first saved to some string buffer,
  checked, converted and only after that will be written to the file.
    
3. Parameters of the file are hardcoded. Its' name, open mode, type
of separator and so on. If you want to change some of them, you'll
have to manually amend code and recompile your program.

To eliminate this drawbacks (well, most of them) I wrote small
library - **[qtcsv][6]**. It has class [**Reader**][7] that can read csv-files.
It has class [**Writer**][12] that can write csv-files. Also it has several
container classes for data that is going to be written to csv-file. Let's
examine it in more detail.

*qtcsv* library have three container classes. [**AbstractData**][8]  is a pure
abstract class that provide interface for a concrete container classes
[**StringData**][9] and [**VariantData**][10]. It has only basics functions
for adding new rows, getting rows values, clearing all data and checking if
container is empty or not:

``` cpp
class AbstractData
{
public:
    explicit AbstractData() {}
    virtual ~AbstractData() {}

    // Add new empty row
    virtual void addEmptyRow() = 0;
    // Add new row with specified values (as strings)
    virtual void addRow(const QStringList& values) = 0;
    // Clear all data
    virtual void clear() = 0;
    // Check if there are any rows
    virtual bool isEmpty() const = 0;
    // Get number of rows
    virtual int rowCount() const = 0;
    // Get values of specified row as list of strings
    virtual QStringList rowValues(const int& row) const = 0;
};
```

As you can see, functions in *AbstractData* are only declared, but not
defined. And class *AbstractData* don't know how "raw" data is actually
saved in container class. It is up to user to define such things in concrete
classes.

First concrete container class in *qtcsv* library is [*StringData*][9]. From
its name you can guess how this container hold data. Yes, in strings. Or to be
more precise - in *QList\<QStringList\>*. *StringData* implements all abstract
functions of *AbstractData* plus it has some container-specific functions
like *insertRow(), replaceRow() and operator<<()*. *StringData* works only with
strings. It accepts new data only in strings, it returns data in strings and
so on. So it is best to use *StringData* if your data is already represented
in strings.

Second concrete container class in *qtcsv* library is [*VariantData*][10]. As
*StringData*, it implements all abstract functions of *AbstractData* plus it
has some container-specific functions. The main difference is that
*VariantData* holds data in [*QVariant*][11] type (*QList\<QList\<QVariant\>\>*).
In Qt *QVariant* is a very specific and useful type. It works like a wrapper for
a many types. You can create *QVariant* variable from *int, string, bool,
double* or Qt-specific classes like *QDate, QByteArray* and many others (see
list of constructors of *QVariant* in [Qt documentation][11]). So if your data
is a set of strings, ints and doubles, it is easier to use *VariantData* as
a container for your data than *StringData*. Because when it will be time to
write your content to file, *VariantData* automatically transform data to
strings.

I see the use of *VariantData* as a solution to the first drawback of the
traditional approach to working with csv-files. With *VariantData* you don't
need additional step of data transformation. In most cases. But if your data use
specific type to store information, *VariantData* won't help here because it
don't know how to transform your specific type to string. It will be your work.

So now we know how to store information in containers of *qtcsv* library. Next
let's see how to write it to csv-file. Here is [*Writer*][12] class:

``` cpp
class Writer
{
public:
    enum WriteMode
    {
        REWRITE = 0,
        APPEND
    };

    // Write data to csv-file
    static bool write(const QString& filePath,
                      const AbstractData& data,
                      const QString& separator = ",",
                      const WriteMode& mode = REWRITE,
                      const QStringList& header = QStringList(),
                      const QStringList& footer = QStringList(),
                      QTextCodec* codec = QTextCodec::codecForLocale());
};
```

*Writer* has only one function - *write()*. This function have many arguments,
but most of them have default values. There are only two arguments that you must
to specify:

  - *filePath* -  this is a string with absolute path to csv-file. Examples of absolute path:
    - in Linux: /home/user/file.csv
    - in Windows: C:\\tmp\file.csv

    There is one more requirement to *filePath*: it must ends with ".csv".

  - *data* - this is a container object, derived from *AbstractData*.

Let's move on to additional arguments:

  - *separator* - it is a symbol that separates elements in a row in csv file;
  - *mode* - this is write flag. If mode is set to WriteMode::APPEND
  and csv-file exist, then new information will be appended to the end of the
  file. If it set to WriteMode::REWRITE and csv-file exist, then all
  information will be written to temporary csv-file and after that destination
  csv-file will be replaced by this temporary file;
  - *header* - strings that will be written at the beginning of the file,
  separated with defined separator;
  - *footer* -  strings that will be written at the end of the file, separated
  with defined separator;
  - *codec* - pointer to the codec object that will be used to write data to
  the file. Use this argument if you want to save csv-file in specific coding
  (like Windows-1251).

That amount of arguments is explained by desire to provide the most flexible
way of working with csv-files. You can change parameters on the fly, no
recompilation (kind of my solution to the third drawback).

*Writer* sends data to file by chunks. It collects several rows from original
data, transform it to string (if necessary), adds separators, new line symbols
and send to *QTextStream* which is then send it to *QFile*. And then cycle repeats
till there is no data left. This approach is very useful when you have big
amount of data. *Writer* will not take much memory so your program will run
smoothly. Also *Writer* provides solution to the 2.2 drawback. It will convert
your data to strings only when it necessary.

Finally we get to the [*Reader*][7]. The purpose of this class is obvious - reading
content of the csv-file. Here is its interface:

``` cpp
class Reader
{
public:
    // Read csv-file and save it's data as strings to QList<QStringList>
    static QList<QStringList> readToList(const QString& filePath,
                        const QString& separator = ",",
                        QTextCodec* codec = QTextCodec::codecForLocale());

    // Read csv-file and save it's data to AbstractData-based container
    // class
    static bool readToData(const QString& filePath,
                        AbstractData& data,
                        const QString& separator = ",",
                        QTextCodec* codec = QTextCodec::codecForLocale());
};
```

*Reader* have two functions. The first - *readToList()* - read the csv-file
into list of strings. Pretty straightforward, just like functions *ReadCSV()*
in the beginning of this post. The second function - *readToData()* - is a little
more interesting. It will read csv-file the same way as the first function,
but it will save read data into *AbstractData*-based container using function 
*addRow(QStringList)*. You can create your own *AbstractData*-based container and
implement this function such a way that it will automatically interpret or
convert this strings the way you want it.

Both of the functions of the *Reader* class have two additional arguments:

  - *separator* - it is a symbol that separates elements in a row in csv file;
  - *codec* - pointer to the codec object that will be used to read data from
  the file. Use this argument if you want to read csv-file that was saved in
  specific coding (like KOI8-R).

Unfortunately *Reader* don't provide solution to drawback 2.1. It reads the
whole file at once and saves all its content to container.

And this is it. I reviewed all public classes of qtcsv library. I don't think
it is perfect - it is definitely not. qtcsv do not eliminate all the drawbacks
I have mentioned. But it provides simple and easy-to-use interface to work with
csv-files. Plus it could be extended to meet your needs.

I also created example project, that use qtcsv library. It's like a quick-start
or playground. I hope it will be useful for your.

https://github.com/iamantony/qtcsv-example.git

[1]: http://matplotlib.org/
[2]: http://en.wikipedia.org/wiki/Comma-separated_values
[3]: http://lmgtfy.com/?q=how+to+create+plots+in+excel
[4]: https://en.wikipedia.org/wiki/Pixel
[5]: https://en.wikipedia.org/wiki/Histogram
[6]: https://github.com/iamantony/qtcsv
[7]: https://github.com/iamantony/qtcsv/blob/master/src/include/reader.h
[8]: https://github.com/iamantony/qtcsv/blob/master/src/include/abstractdata.h
[9]: https://github.com/iamantony/qtcsv/blob/master/src/include/stringdata.h
[10]: https://github.com/iamantony/qtcsv/blob/master/src/include/variantdata.h
[11]: http://doc.qt.io/qt-5/qvariant.html
[12]: https://github.com/iamantony/qtcsv/blob/master/src/include/writer.h
