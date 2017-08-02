# 醸造に関するメモ


## ちゃんとした醸造
```json
"brewable" : {
  "time" : 72000,
  "results" : [ "mead", "yeast" ]
}
```
timeの定義はターン数なので、`"time" : 1`は6秒になる。上の例だと`72,000 * 6 = 432,000秒 = 5日`になる。

ところが、醸造時間には生成した**ワールドの季節の日数で補正がかかる。**具体的には`* ( calendar::season_length() / 14.0 )`される。一季節をデフォルト(14日)の倍である28日でやっていると、醸造にも倍の時間が掛かることになる。


* `brewable->time`の定義  
https://github.com/CleverRaven/Cataclysm-DDA/blob/00af4ca46259f2601bb4df67647662ac9ff291c1/src/itype.h#L148-L154
```c++
struct islot_brewable {
    /** What are the results of fermenting this item? */
    std::vector<std::string> results;

    /** How many turns for this brew to ferment */
    int time = 0;
};
```

* `brewing_time`にかかる補正  
https://github.com/CleverRaven/Cataclysm-DDA/blob/00af4ca46259f2601bb4df67647662ac9ff291c1/src/item.cpp#L3007-L3010
```c++
int item::brewing_time() const
{
    return ( is_brewable() ? type->brewable->time : 0 ) * ( calendar::season_length() / 14.0 );
}
```


## delayed_transformを使って醸造っぽく見せる方法
```json
"use_action": {
  "type": "delayed_transform",
  "transform_age": 72000,
  "not_ready_msg": "まだだよ",
  "msg": "できたよ",
  "moves": 0,
  "target": "id_of_target",
  "container": "id_of_container_of_taget"
}
```
`transform_age`は例によってターン数なので、`"transform_age" : 1`は6秒になる。なお、季節の日数による補正は掛からない。

* `delayed_transform_iuse`の定義  
https://github.com/CleverRaven/Cataclysm-DDA/blob/dbf94ea32432320fc874237d93965c069fb674f3/src/iuse_actor.h#L232-L260
```c++
/**
 * This is a @ref iuse_transform that uses the age of the item instead of a counter.
 * The age is calculated from the current turn and the birthday of the item.
 * The player has to activate the item manually, only when the specific
 * age has been reached, it will transform.
 */
class delayed_transform_iuse : public iuse_transform
{
    public:
        /**
         * The minimal age of the item (in turns) to allow the transformation.
         */
        int transform_age = 0;
        /**
         * Message to display when the user activates the item before the
         * age has been reached.
         */
        std::string not_ready_msg;

        /** How much longer (in turns) until the transformation can be done, can be negative. */
        int time_to_do( const item &it ) const;

        delayed_transform_iuse( const std::string &type = "delayed_transform" ) : iuse_transform( type ) {}

        ~delayed_transform_iuse() override;
        void load( JsonObject &jo ) override;
        long use( player *, item *, bool, const tripoint& ) const override;
        iuse_actor *clone() const override;
};
```


# 付録

## ターン換算表
Turn | 時間
-:|-:
1 | 6秒
10 | 1分
600 | 1時間
14,400 | 1日

## リンク
- [JSON_INFO.md](https://github.com/CleverRaven/Cataclysm-DDA/blob/master/doc/JSON_INFO.md)
