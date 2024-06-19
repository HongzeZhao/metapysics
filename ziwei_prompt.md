``` javascript
const { astro } = require('iztro');

const allMutagens = ['科', '权', '禄', '忌'];

/**
 * 计算时辰序号
 * @param {string} hourMin format HH:mm
 * @returns {int} 早子时为0，返回值范围是 0 ~ 12
 */
function getHourIndexFromTime(hourMin) {
    const parts = hourMin.split(':');
    if (parts.length === 2) {
        const hour = parseInt(parts[0]);
        return hour === 0 ? 0 : parseInt((hour + 1) / 2);
    }
    return -1;
}

function getCurrentDateStr() {
    let date = new Date();
    let year = date.getFullYear();
    let month = (date.getMonth() + 1).toString().padStart(2, '0');
    let day = date.getDate().toString().padStart(2, '0');
    return year + '-' + month + '-' + day;
}

function getCurrentHourIndex() {
    let date = new Date();
    let hour = date.getHours();
    return hour === 0 ? 0 : parseInt((hour + 1) / 2);
}

/**
 * 计算四化
 * @param {string} horoscope 根据当前时间计算的星象
 * @param {string} palace 宫位名称，例如'夫妻'
 * @param {string} scope 运限名称，origin：本命星曜；decadal：大限星曜；yearly：流年星曜
 */
function getMutagen(horoscope, palace, scope) {
    const results = [];
    allMutagens.forEach(mutagen => {
        if (horoscope.hasHoroscopeMutagen(palace, scope, mutagen)) {
            results.push('化' + mutagen);
        }
    });
    return results.join('');
}

function getAllMutagens(horoscope, palace) {
    let mutagens = [];

    // 大限四化
    const decadalMutagen = getMutagen(horoscope, palace, 'decadal');
    if (decadalMutagen.length > 0) {
        mutagens.push('大限星曜' + decadalMutagen);
    }

    // 流年四化
    const yearlyMutagen = getMutagen(horoscope, palace, 'yearly');
    if (yearlyMutagen.length > 0) {
        mutagens.push('流年星曜' + yearlyMutagen);
    }
    return mutagens.join(',');
}

/**
 * 根据阳历的生日、性别生成紫微斗数星盘的描述
 * @param {string} dateStr format yyyy-MM-dd
 * @param {string} timeStr format HH:mm
 * @param {string} gender  format 男/女
 * @returns 
 */
function getZiweiPlateDesc(dateStr, timeStr, gender) {
    const astrolabe = astro.bySolar(dateStr, getHourIndexFromTime(timeStr), gender, true, 'zh-CN');

    function getStarListInfo(stars) {
        if (stars.length <= 0) return '为空';
        let infos = [];
        stars.forEach(s => {
            let info = s.name;
            if (s.brightness) {
                info += `(${s.brightness})`;
            }
            if (s.mutagen) {
                info += `化${s.mutagen}`;
            }
            infos.push(info);
        });
        return '为' + infos.join('、');
    }

    let desc = `我的生辰八字为${astrolabe.chineseDate}，${astrolabe.gender}性，${astrolabe.sign}，属${astrolabe.zodiac}，${astrolabe.fiveElementsClass}，命主${astrolabe.soul}，身主${astrolabe.body}。\n`;

    desc += "# 紫微斗数星盘信息：\n";
    astrolabe.palaces.forEach(p => {
        desc += `## ${p.name}宫（${p.heavenlyStem}${p.earthlyBranch}宫）`;
        if (p.isBodyPalace) {
            desc += '，身宫';
        }
        if (p.isOriginalPalace) {
            desc += '，来因宫';
        }
        desc += '\n';

        majorStars = getStarListInfo(p.majorStars);
        minorStars = getStarListInfo(p.minorStars);
        adjectiveStars = getStarListInfo(p.adjectiveStars);
        desc += `主星${majorStars}，辅星${minorStars}，杂曜${adjectiveStars}。\n`;
        desc += `长生十二神为${p.changsheng12}，博士十二神为${p.boshi12}，将前十二神为${p.jiangqian12}，岁前十二神为${p.suiqian12}。\n`;
        desc += `大限干支为${p.decadal.heavenlyStem}${p.decadal.earthlyBranch}，${p.decadal.range[0]}岁到${p.decadal.range[1]}岁。\n`;
        ages = [];
        p.ages.forEach(age => ages.push(`${age}岁`));
        desc += '小限为：' + ages.join(',') + '。\n';

        // 流年信息
        const horoscope = astrolabe.horoscope();
        desc += '运化流年相关：\n';
        
        const mutagenInfo = getAllMutagens(horoscope, p.name);
        if (mutagenInfo.length > 0) {
            desc += mutagenInfo + '。\n';
        }

        astrolabe.palaces.forEach(fromPalace => {
        allMutagens.forEach(mutagen => {
                if (fromPalace.fliesTo(p.name, mutagen)) {
                    desc += `${fromPalace.name}化${mutagen}入${p.name}\n`;
                }
            });
        });

        allMutagens.forEach(mutagen => {
            if (p.selfMutaged(mutagen)) {
                desc += `${p.name}自化${mutagen}\n`;
            }
        });

        desc += '\n\n';
    });

    return desc;

}

//desc += '请根据紫微斗数和以上的星盘信息，逐一推算我的相貌、性格、家庭、父母、子女、配偶、财运、事业、交友、健康、置业等各方面信息，并在以上各方面给出尽可能详尽的建议。此外，还要预测我今年各方面的运势如何。';

//console.log(desc);

console.log(getZiweiPlateDesc('1995-08-14', '05:15', '女'));

//console.log(getHourFromTime("23:45"));

console.log(getCurrentHourIndex());
```
