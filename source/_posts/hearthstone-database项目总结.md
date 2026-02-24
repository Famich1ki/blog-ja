---
title: Hearthstone Database Summary
date: 2024-08-27 14:19:50
tags:
  - Spring Boot
  - Java
  - MyBatis
  - MySQL
categories:
  - Hands-on Project
cover: https://pics.findfuns.org/hearthstone.jpg
---



## Preface

This is a full-stack project based on a front-end and back-end separation architecture. The back-end tech stack includes Spring Boot + MyBatis + MySQL. The front-end is built with Vue 3. After completion, the project was deployed on AWS cloud services, running on Ubuntu, with Nginx configured as a reverse proxy.

The entire project—from collecting the dataset, cleaning the data, integrating the database, to finally deploying it to the server—took approximately 10 days. The actual front-end and back-end development did not take very long, nor did it involve particularly complex technologies. Most of the time was spent on database cleaning and integration.

The metadata came from a third-party Hearthstone website API (many thanks to them). However, after retrieving the metadata, it could not be used directly due to dirty data, redundant entries, and formatting inconsistencies. Fixing these issues took about 3–4 days.

This article summarizes and reviews the issues encountered and the major workload involved throughout the development process.

Below is a screenshot of the IDEA project structure:

```html
<img src="https://pics.findfuns.org/backend-overview.png" alt="back" style="zoom:33%;" />
```

The entire back-end project follows the classic Spring MVC architecture: Controller–Service–Mapper layers. POJO classes and custom `TypeHandler`s were also designed. The custom `TypeHandler` is used to handle mappings between database query results and Java objects when default mappings are insufficient.

---

## POJO

Let’s first talk about the POJO classes. To accommodate different card types and their various attributes, the POJO classes were carefully designed, including inheritance relationships, enum classes, and data types.

```html
<img src="https://pics.findfuns.org/pojo-hierarchical-diagram.png" alt="z" style="zoom:33%;" />
```

The `Card` class is the base class for all cards. Every card type extends the `Card` class. It contains the fundamental attributes shared by all cards.

```java
package com.zzb.hearthstoneDB.pojo;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Card {

    private String id; // Unique ID of each card

    private String name; // Card name

    private Integer cost; // Mana cost

    private CardClass cardClass; // MAGE, DRUID, PRIEST ...

    private Integer cardSet; // Integer representing a specific expansion set

    private String rule; // Card description
}
```

There are six subclasses of `Card`: `Spell`, `Hero`, `HeroPower`, `Location`, `Weapon`, and `Minion`, corresponding to six card types: spells, hero cards, hero powers, locations, weapons, and minions.

For example, the design of the `Minion` class:

```java
package com.zzb.hearthstoneDB.pojo;

@Data
@NoArgsConstructor
public class Minion extends Card {
    public Minion(String id, String name, Integer cost, CardClass cardClass, Integer cardSet,
                  String rule, Integer attack, Integer health, Rarity rarity, Race race, String flavor) {
        super(id, name, cost, cardClass, cardSet, rule);
        this.attack = attack;
        this.health = health;
        this.rarity = rarity;
        this.race = race;
        this.flavor = flavor;
    }

    private Integer attack; // Attack value

    private Integer health; // Health value

    private Rarity rarity; // Rarity

    private Race race; // Tribe

    private String flavor; // Flavor text
}
```

Several attributes such as `Rarity`, `Race`, `CardClass`, and `SpellSchool` have a fixed set of possible values, making them well-suited for enum classes.

For example, the `SpellSchool` enum:

```java
package com.zzb.hearthstoneDB.pojo;

public enum SpellSchool {

    ARCANE, // Arcane
    FIRE, // Fire
    FROST, // Frost
    NATURE, // Nature
    SHADOW, // Shadow
    HOLY, // Holy
    FEL // Fel
}
```

---

## Controller

The Controller layer is responsible for route mapping, extracting parameters from URLs, and passing them to the Service layer.

```java
package com.zzb.hearthstoneDB.controller;

@RestController
@RequestMapping("cards/api")
@CrossOrigin
public class CardController {

    private final CardService cardService;

    @Autowired
    public CardController(CardService cardService) {
        this.cardService = cardService;
    }
}
```

`@RestController` is a composite annotation that includes both `@Controller` and `@ResponseBody`.  
`@Controller` marks the class as a Spring MVC controller, and `@ResponseBody` ensures that the return value is written directly to the HTTP response body in JSON format.

`@Autowired` automatically injects the Service component.

`@RequestMapping` defines the base route. Note that **the base route does not start with '/'**, which differs from method-level routes.

A typical controller method looks like this:

```java
@GetMapping("/minion")
public List<Card> selectMinions(
  @RequestParam(value = "name", required = false) String name, 
  @RequestParam(value = "cost", required = false) Integer cost,
  @RequestParam(value = "attack", required = false) Integer attack, 
  @RequestParam(value = "health", required = false) Integer health,
  @RequestParam(value = "rarity", required = false) String rarity, 
  @RequestParam(value = "race", required = false) String race,
  @RequestParam(value = "cardClass", required = false) String cardClass,
  @RequestParam(value = "cardSet", required = false) Integer cardSet,
  @RequestParam(value = "rule", required = false) String rule) {

    return cardService
      .selectMinions(name, cost, attack, health, rarity, race, cardClass, cardSet, rule);
}
```

`@GetMapping` specifies that the method handles GET requests and defines the corresponding route.

`@RequestParam` extracts query parameters from a GET request, such as:

```
localhost:8080/minion?cost=9&rarity=RARE
```

The `required` parameter defaults to `true`. If not provided, Spring throws a `MissingServletRequestParameterException` and returns HTTP 400. Setting it to `false` allows null values.

---

## Service

The Service layer packages the parameters from the Controller into corresponding POJO objects and passes them to the Mapper layer for querying.

```java
package com.zzb.hearthstoneDB.service;

@Service
public class CardService {

    private final CardMapper cardMapper;

    @Autowired
    public CardService(CardMapper cardMapper) {
        this.cardMapper = cardMapper;
    }
}
```

Example Service method:

```java
public List<Card> selectMinions(String name, Integer cost, Integer attack, 
                                Integer health, String rarity, String race, 
                                String cardClass,Integer cardSet, String rule) {

    Minion minion = new Minion(null, name, cost, 
                               cardClass == null ? null : CardClass.valueOf(cardClass),
                               cardSet, rule, attack, health, 
                               rarity == null ? null : Rarity.valueOf(rarity), 
                               race == null ? null : Race.valueOf(race), null);

    return cardMapper.selectMinion(minion);
}
```

---

## Mapper

The Model layer uses MyBatis as the ORM framework. In MyBatis, the Mapper interface defines SQL mappings.

XML configuration files are placed under the `resources` directory, and **the folder structure must exactly match the Java package structure**.

Example Mapper interface:

```java
package com.zzb.hearthstoneDB.mapper;

@Mapper
public interface CardMapper {

    List<Card> selectCards(@Param("card_query") Card card);

    String selectCardById(String id);

    List<Card> selectMinion(@Param("minion") Minion minion);

    List<Card> selectSpell(@Param("spell") Spell spell);

    List<Card> selectWeapon(@Param("weapon") Weapon weapon);

    List<Card> selectHero(@Param("hero") Hero hero);

    List<Card> selectHeroPower(@Param("heroPower") HeroPower heroPower);

    List<Card> selectLocation(@Param("location") Location location);
}
```

---

### XML Basic Structure

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

```xml
<mapper namespace="com.zzb.hearthstoneDB.mapper.CardMapper">
</mapper>
```

---

### resultMap Example

```xml
<resultMap id="cardResultMap" type="com.zzb.hearthstoneDB.pojo.Card">
    <result property="id" column="id"/>
    <result property="name" column="name"/>
    <result property="cost" column="cost"/>
    <result property="cardClass" column="card_class"
            typeHandler="com.zzb.hearthstoneDB.typeHandler.CardClassTypeHandler"/>
    <result property="cardSet" column="card_set"/>
    <result property="rule" column="rule"/>
</resultMap>
```

---

### Dynamic SQL Example

```xml
<select id="selectMinion" resultMap="minionResultMap">
    SELECT minion.id, name, cost, attack, health, rarity, race, card_class,
           card_set_id.id as card_set, rule, flavor
    FROM minion
    JOIN card_set_id ON minion.card_set = card_set_id.card_set
    <where>
        AND collectible = 1

        <if test="minion.name != null">
            AND name LIKE CONCAT('%', #{minion.name}, '%')
        </if>

        <if test="minion.cost != null and minion.cost &lt; 10">
            AND cost = #{minion.cost}
        </if>

        <if test="minion.cost != null and minion.cost == 10">
            AND cost &gt;= #{minion.cost}
        </if>

        <if test="minion.cardClass != null">
            AND card_class = #{minion.cardClass}
        </if>

        <if test="minion.cardSet != null">
            AND card_set_id.id = #{minion.cardSet}
        </if>

        <if test="minion.rule != null">
            AND rule LIKE CONCAT('%', #{minion.rule}, '%')
        </if>

        <if test="minion.attack != null">
            AND attack = #{minion.attack}
        </if>

        <if test="minion.health != null">
            AND health = #{minion.health}
        </if>

        <if test="minion.rarity != null">
            AND rarity = #{minion.rarity}
        </if>

        <if test="minion.race != null">
            AND race = #{minion.race}
        </if>
    </where>
</select>
```

---

## Database Design

```html
<img src="https://pics.findfuns.org/hearthstone-database-diagram.png" style="zoom:33%;" />
```

The database consists of six tables, each corresponding to a specific card type. All tables use `id` as the primary key.