头文件同时被mm与cpp文件引用时，可以利用宏__OBJC__来分离头文件里的oc与c++声明：
```cpp
#include "cocos2d.h"

#ifdef __OBJC__
@class CCAVPlayerImpl;
#else
class CCAVPlayerImpl;
#endif

class CCAVPlayer
{
private:
    CCAVPlayerImpl* _impl;
public:
    void init(const std::string& url);
    void play();
    ~CCAVPlayer();
};
```