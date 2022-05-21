## QRSpeed词库BSH自适应图文

---

支持设置背景颜色，文字边距，字体大小，字体下划线，加粗等功能…

#### 文档目录
* [java代码](#java代码)
* [词库代码](#词库代码)

##### java代码
```java
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Typeface;
import android.text.TextPaint;
import java.io.FileNotFoundException;
import java.io.File;
import java.io.IOException;
import java.io.FileOutputStream;
import android.content.SharedPreferences;
import android.content.Context;

/**
 * Created by Yingy.
 * Name: 自适应图文
 * QQ: 2665501650
 */

public final String CONFIG_NAME = "yingy_image";


public String imagettftext(String content)
{
	String ttf= "/mnt/sdcard/QR/QRDic/BSH/default.ttf";
	int textSize = 50;
	String[] split = content.split("\\[换行\\]");
	int maxStringLength = 0;
	TextPaint textPaint = new TextPaint();
	textPaint.setTextSize(textSize);
	if(getBool("bold", false)) textPaint.setFakeBoldText(true); // 加粗
	if(getBool("underline", false)) textPaint.setUnderlineText(true); // 加下划线
	if(getBool("antialias", false)) textPaint.setAntiAlias(true); // 抗锯齿
	if(getBool("dither", false)) textPaint.setDither(true); // 抖动处理
	if(getBool("strikethru", false)) textPaint.setStrikeThruText(true); //删除线
	textPaint.setTypeface(Typeface.createFromFile(new File(ttf)));
	for(String str : split)
	{
		int length = Math.round(textPaint.measureText(str));
		if(length > maxStringLength)
		{
			maxStringLength = length;
		}
	}
	int padding = getInt("padding", 100);
	int width = (padding * 2) + (maxStringLength);
	int height = (padding * 2) + (split.length * textSize);
	Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
	Canvas canvas = new Canvas(bitmap);
	String background = getString("background", "");
	if(!background.isEmpty())
		canvas.drawColor(Color.parseColor(background));
	else
		canvas.drawColor(Color.WHITE);
	int y = padding;
	for(String string : split)
	{
		String color = "#";
		String[] str= new String[]{"0","1","2","3","4","5","6","7","8","9","A","B","C","D","E","F"};
		for (int i=1;i <= 6;i++) {
			int random = 0 + (int)(Math.random() * (str.length - 0));
			color += str[random];
		}
		textPaint.setColor(Color.parseColor(color));
		canvas.drawText(string, padding, y += textSize, textPaint);
	}
	canvas.save();
	canvas.restore();
	String path = saveBitmap(bitmap);
	bitmap.recycle();
	return path;
}

public String saveBitmap(Bitmap bitmap)
{
	String path="/mnt/sdcard/QR/QRDic/data/cache/"+ System.currentTimeMillis() +".png";
	File file = new File(path);
	if (!file.exists()) {
		file.getParentFile().mkdirs();
		try {
			file.createNewFile();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	try {
		FileOutputStream out = new FileOutputStream(file);
		bitmap.compress(Bitmap.CompressFormat.PNG, 100, out);
		out.flush();
		out.close();
	} catch (FileNotFoundException e) {
		e.printStackTrace();
	} catch (IOException e) {
		e.printStackTrace();
	}
	return path;
}

public boolean putBool(String key, String val)
{
	SharedPreferences preferences = context.getSharedPreferences(CONFIG_NAME, Context.MODE_PRIVATE);
	SharedPreferences.Editor editor = preferences.edit();
	editor.putBoolean(key, Boolean.valueOf(val));
	return editor.commit();
}

public boolean putString(String key, String val)
{
	SharedPreferences preferences = context.getSharedPreferences(CONFIG_NAME, Context.MODE_PRIVATE);
	SharedPreferences.Editor editor = preferences.edit();
	editor.putString(key, val);
	return editor.commit();
}

public boolean putInt(String key, String val)
{
	SharedPreferences preferences = context.getSharedPreferences(CONFIG_NAME, Context.MODE_PRIVATE);
	SharedPreferences.Editor editor = preferences.edit();
	editor.putInt(key, Integer.parseInt(val));
	return editor.commit();
}

public boolean getBool(String key, boolean defaultVal)
{
	SharedPreferences preferences = context.getSharedPreferences(CONFIG_NAME, Context.MODE_PRIVATE);
	return preferences.getBoolean(key, defaultVal);
}

public String getString(String key, String defaultVal)
{
	SharedPreferences preferences = context.getSharedPreferences(CONFIG_NAME, Context.MODE_PRIVATE);
	return preferences.getString(key, defaultVal);
}

public int getInt(String key, int defaultVal)
{
	SharedPreferences preferences = context.getSharedPreferences(CONFIG_NAME, Context.MODE_PRIVATE);
	return preferences.getInt(key, defaultVal);
}
```

##### 词库代码
```text
图文 ?(.*)
success±img=$BSH Image-text.java imagettftext %括号1%$±

(开启|关闭)(加粗|下划线|抗锯齿|删除线|抖动处理)
B:
K:
如果:%括号1%==开启
B:true
如果尾
如果:%括号1%==关闭
B:false
如果尾
如果:%括号2%==加粗
K:bold
如果尾
如果:%括号2%==下划线
K:underline
如果尾
如果:%括号2%==抗锯齿
K:antialias
如果尾
如果:%括号2%==删除线
K:strikethru
如果尾
如果:%括号2%==抖动处理
K:dither
如果尾
如果:%K%==||%B%==
错误
返回
如果尾
R:$BSH Image-text.java putBool %K% %B%$
如果:%R%==true
设置成功
返回
如果尾
设置失败

设置边距 ?([0-9]+)
R:$BSH Image-text.java putInt padding %括号1%$
如果:%R%==true
设置成功
返回
如果尾
设置失败

设置背景 ?(.*)
R:$BSH Image-text.java putString background %括号1%$
如果:%R%==true
设置成功
返回
如果尾
设置失败
```
