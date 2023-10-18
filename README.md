# Metin2 Mining Bot

**This project was made only for educational and learning purposes.**

Project built using .NET desktop application and EmguCV library (.NET wrapper for OpenCV image processing library). Main functionality of this project is to enable automatic mining of resources in game Metin2. This bot does not use code injection but relies on image processing algorithms, so its not dependent on any libraries used by game client.

### How mining works
Character needs to stand next to ore vein with pickaxe equipped. Mining is initiated by left clicking on ore vein. Mining process takes about 5-20 seconds after which based on the pickaxe level and mining skill, certain amount of ore is dropped on the ground. There is a chance that no ore will be dropped if characters skill is not high enough. After picking up the ore, mining process can start again.
After certain amount of time or mining cycles, ore vein will disappear and will respawn after few minutes. New location and respawn time varies greatly between Metin2 servers. This bot was made on Czech private server Stellaria, where ores respawn after approximately 1 minute at the same place.

### User interface

<img src="https://github.com/AndrejVysinsky/metin2-miner-readme/assets/59775817/30242b30-3faf-4c25-8527-137d2221f5f9" width="400">
 
1.	Show/hide configuration windows needed for bot to function properly.
2.	License information got from backend. Without active license, bot won’t start.
3.	Status information. When running provides additional information.
4.	Number of mining bots. Represents on how many windows or instances of game user wants to use this bot.
5.	Controls to start/stop the program.

### Configuration

<img src="https://github.com/AndrejVysinsky/metin2-miner-readme/assets/59775817/ed28e7a5-f23a-4f2f-8016-098494e6b0de" width="600">

Each instance of mining bot has 2 configuration windows, mining point and loot window. Both windows are movable and resizable to accommodate for different screen resolution and positions of UI elements.

Mining point serves as a clickable point in the centre of which the ore is located. This point is clicked automatically every mining cycle. We know that ore respawns always at the same place which we can take the advantage of. When ore vein disappears, we would have been clicking on the ground and eventually move away and get stuck. However, under characters feet there is a small space which prevents character from moving forward but allows us to click at ore vein. This way we won’t wander away even if the ore vein disappears.

Loot window detects any change of its contents. Every picked up item shows in this part of the screen. On every update (1-2 seconds) bot tries to pick up items from the ground and subsequently checks any changes inside the loot window. Each check compares it with default version and if there is significant  difference, loot was collected which means mining cycle ended and we can start the next one.

Comparing two images/finding one image in another:
```
public static SearchResult SearchForImage(string imageToSearchFor, string imageToSearchThrough = defaultPath, double matchTreshold = 0.85)
{
    Image<Bgr, byte> originalImage = new(imageToSearchThrough);
    Image<Bgr, byte> templateImage = new(imageToSearchFor);

    Image<Gray, float> resultImage = originalImage.MatchTemplate(templateImage, TemplateMatchingType.CcoeffNormed);
    resultImage.MinMax(out _, out double[] maxValues, out _, out Point[] maxLocations);

    var searchResult = new SearchResult()
    {
        Width = templateImage.Width,
        Height = templateImage.Height,
        MaxMatch = maxValues[0]
    };

    originalImage.Dispose();
    templateImage.Dispose();

    Point matchLocation = maxLocations[0];
    if (maxValues[0] >= matchTreshold)
    {
        searchResult.Point = matchLocation;
        return searchResult;
    }
    else
    {
        searchResult.Point = SearchResult.Empty;
        return searchResult;
    }
}
```

### License
When application starts, client must authorize against backend database. Client is identified mainly using UUID and if exists in the database he gets as a response license information. If license is expired, bot will not start. For communication with backend, Bearer authentication method is used.

### Captcha solver

<img src="https://github.com/AndrejVysinsky/metin2-miner-readme/assets/59775817/71d808c4-77b3-42b6-8fc9-463a3eea0860" width="200">

Server Stellaria uses captcha system to prevent players from using auto clickers or some other automation. Captcha system consists of image with 3 random numbers and numpad to enter the numbers from image. Captcha is displayed every 15 minutes and failing to complete it disconnects player from server.

Captcha solving is not happening in this application as OCR algorithms by default are not built for it and were failing reading the numbers most of the time. Instead, application sends image to backend where we are calling AZCaptcha API (https://azcaptcha.com/) which solves the image for us. Result is sent back to the client and numbers are automatically inputted.

### Showcase

https://mega.nz/file/jihDSRTC#xC_Qr3YUiO6F5uZrA-TtyjpjDOeQtfpAPa-XVCdc8i4
