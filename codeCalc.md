import React, { useState, useEffect } from 'react';  
import { Calculator, Wallet, ArrowRightLeft, Coins, CheckCircle2, Circle, TrendingUp } from 'lucide-react';

const TelegramStarsCalculator \= () \=\> {  
 // \--- Глобальные параметры (Константы бизнеса) \---  
 const \[sellingPrice, setSellingPrice\] \= useState(1.29); // Почем продаем (RUB)  
  // Параметры Fragment (курс обмена TON \-\> Stars)  
 const \[fragmentBatchSize, setFragmentBatchSize\] \= useState(50);  
 const \[fragmentBatchCost, setFragmentBatchCost\] \= useState(0.4915);

 // \--- Режим расчета (по звездам или по бюджету TON) \---  
 const \[calcMode, setCalcMode\] \= useState('stars'); // 'stars' | 'ton'  
 const \[inputVolume, setInputVolume\] \= useState(1000); // Количество звезд или TON

 // \--- Источники закупки (Курсы TON) \---  
 // Храним курсы отдельно, чтобы их редактировать  
 const \[rates, setRates\] \= useState({  
   p2p: 125.0,  
   bestchange: 120.5,  
   exchange: 116.3  
 });

 // Выбранный активный источник для итога  
 const \[selectedSourceId, setSelectedSourceId\] \= useState('p2p');

 // Результаты вычислений  
 const \[calculations, setCalculations\] \= useState(\[\]);  
 const \[summary, setSummary\] \= useState(null);

 useEffect(() \=\> {  
   // 1\. Базовый курс: сколько TON стоит 1 звезда  
   const tonPerStar \= fragmentBatchCost / fragmentBatchSize;  
   // Обратный: сколько звезд в 1 TON  
   const starsPerTon \= fragmentBatchSize / fragmentBatchCost;

   // 2\. Определяем объемы продаж и закупок в зависимости от режима  
   let volumeStars \= 0;  
   let volumeTon \= 0;

   if (calcMode \=== 'stars') {  
     volumeStars \= inputVolume;  
     volumeTon \= inputVolume \* tonPerStar;  
   } else {  
     volumeTon \= inputVolume;  
     volumeStars \= inputVolume \* starsPerTon;  
   }

   // 3\. Данные источников  
   const sourcesData \= \[  
     { id: 'p2p', name: 'P2P (Wallet)', rate: rates.p2p, desc: 'Покупка внутри Telegram' },  
     { id: 'bestchange', name: 'Bestchange', rate: rates.bestchange, desc: 'Внешние обменники' },  
     { id: 'exchange', name: 'Биржа (OKX/Bybit)', rate: rates.exchange, desc: 'Спотовая торговля' },  
   \];

   // 4\. Считаем экономику для каждого источника  
   const calculatedResults \= sourcesData.map(source \=\> {  
     // Себестоимость всей партии  
     const totalCostRub \= volumeTon \* source.rate;  
      
     // Себестоимость 1 звезды  
     const costPerStar \= source.rate \* tonPerStar;  
      
     // Выручка  
     const totalRevenue \= volumeStars \* sellingPrice;  
      
     // Прибыль  
     const totalProfit \= totalRevenue \- totalCostRub;  
     const profitPerStar \= sellingPrice \- costPerStar;  
      
     // Маржа  
     const margin \= (profitPerStar / sellingPrice) \* 100;

     return {  
       ...source,  
       volumeStars,  
       volumeTon,  
       totalCostRub,  
       costPerStar,  
       totalRevenue,  
       totalProfit,  
       profitPerStar,  
       margin  
     };  
   });

   setCalculations(calculatedResults);

   // 5\. Находим выбранный итог  
   const active \= calculatedResults.find(r \=\> r.id \=== selectedSourceId) || calculatedResults\[0\];  
   setSummary(active);

 }, \[sellingPrice, fragmentBatchSize, fragmentBatchCost, calcMode, inputVolume, rates, selectedSourceId\]);

 // Хендлер изменения курсов  
 const handleRateChange \= (id, newRate) \=\> {  
   setRates(prev \=\> ({ ...prev, \[id\]: Number(newRate) }));  
 };

 return (  
   \<div className\="min-h-screen bg-slate-50 font-sans text-slate-800 pb-10"\>  
      
     {/\* Шапка: Итоговый результат (Всегда перед глазами) \*/}  
     \<div className\="bg-white border-b border-slate-200 sticky top-0 z-10 shadow-sm"\>  
       \<div className\="max-w-5xl mx-auto p-4 md:px-6"\>  
         \<div className\="flex flex-col md:flex-row justify-between items-center gap-4"\>  
           \<div\>  
             \<div className\="text-xs text-slate-500 uppercase tracking-wider font-semibold"\>Ваш результат ({summary?.name})\</div\>  
             \<div className\="flex items-baseline gap-2"\>  
               \<h1 className\={\`text-3xl font-bold ${summary?.totalProfit \>= 0 ? 'text-emerald-600' : 'text-red-500'}\`}\>  
                 {summary?.totalProfit.toLocaleString('ru-RU', { maximumFractionDigits: 0 })} ₽  
               \</h1\>  
               \<span className\={\`text-sm font-medium px-2 py-0.5 rounded ${summary?.totalProfit \>= 0 ? 'bg-emerald-100 text-emerald-700' : 'bg-red-100 text-red-700'}\`}\>  
                 Чистая прибыль  
               \</span\>  
             \</div\>  
           \</div\>  
            
           \<div className\="flex gap-6 text-sm items-center"\>  
              \<div className\="text-right"\>  
                \<div className\="text-slate-400 text-xs uppercase tracking-wide"\>Безубыток\</div\>  
                \<div className\="font-bold text-lg text-amber-600 border-b border-dashed border-amber-300 inline-block" title\="Минимальная цена продажи"\>  
                  {summary?.costPerStar.toFixed(2)} ₽  
                \</div\>  
              \</div\>

              \<div className\="text-right border-l pl-6 border-slate-100"\>  
                \<div className\="text-slate-400 text-xs uppercase tracking-wide"\>Маржа\</div\>  
                \<div className\={\`font-bold text-lg ${summary?.margin \>= 0 ? 'text-emerald-600' : 'text-red-500'}\`}\>  
                  {summary?.margin.toFixed(1)}%  
                \</div\>  
              \</div\>  
               
              \<div className\="text-right border-l pl-6 border-slate-100 hidden sm:block"\>  
                \<div className\="text-slate-400 text-xs uppercase tracking-wide"\>Выручка\</div\>  
                \<div className\="font-bold text-lg text-slate-700"\>  
                  {summary?.totalRevenue.toLocaleString('ru-RU', { maximumFractionDigits: 0 })} ₽  
                \</div\>  
              \</div\>  
           \</div\>  
         \</div\>  
       \</div\>  
     \</div\>

     \<div className\="max-w-4xl mx-auto p-4 md:p-6 space-y-6"\>

       {/\* 1\. Блок: ПАРАМЕТРЫ ФРАГМЕНТА И ПРОДАЖИ \*/}  
       \<div className\="grid grid-cols-1 md:grid-cols-2 gap-4"\>  
         {/\* Фрагмент \*/}  
         \<div className\="bg-white p-5 rounded-xl border border-slate-200 shadow-sm"\>  
           \<h3 className\="font-bold text-slate-700 mb-4 flex items-center gap-2"\>  
             \<Coins className\="w-5 h-5 text-blue-500" /\>  
             Курс Fragment  
           \</h3\>  
           \<div className\="flex gap-2 items-end"\>  
             \<div className\="flex-1"\>  
               \<label className\="text-xs text-slate-400 block mb-1"\>Звезд\</label\>  
               \<input  
                 type\="number"  
                 value\={fragmentBatchSize}  
                 onChange\={(e) \=\> setFragmentBatchSize(Number(e.target.value))}  
                 className\="w-full p-2 bg-slate-50 border border-slate-200 rounded-lg font-medium"  
               /\>  
             \</div\>  
             \<div className\="pb-3 text-slate-400"\>=\</div\>  
             \<div className\="flex-1"\>  
               \<label className\="text-xs text-slate-400 block mb-1"\>TON\</label\>  
               \<input  
                 type\="number"  
                 value\={fragmentBatchCost}  
                 onChange\={(e) \=\> setFragmentBatchCost(Number(e.target.value))}  
                 className\="w-full p-2 bg-slate-50 border border-slate-200 rounded-lg font-medium"  
               /\>  
             \</div\>  
           \</div\>  
           \<div className\="mt-2 text-xs text-blue-600 bg-blue-50 px-2 py-1 rounded inline-block"\>  
             Себестоимость: {(fragmentBatchCost/fragmentBatchSize).toFixed(4)} TON / звезда  
           \</div\>  
         \</div\>

         {/\* Продажа \*/}  
         \<div className\="bg-white p-5 rounded-xl border border-indigo-100 ring-1 ring-indigo-50 shadow-sm"\>  
           \<h3 className\="font-bold text-slate-700 mb-4 flex items-center gap-2"\>  
             \<Wallet className\="w-5 h-5 text-indigo-500" /\>  
             Параметры продажи  
           \</h3\>  
           \<div\>  
             \<label className\="text-xs text-slate-500 block mb-1 uppercase font-semibold"\>Цена продажи (за звезду)\</label\>  
             \<div className\="relative"\>  
               \<input  
                 type\="number"  
                 value\={sellingPrice}  
                 onChange\={(e) \=\> setSellingPrice(Number(e.target.value))}  
                 className\="w-full p-2 pl-3 pr-10 border border-indigo-200 rounded-lg text-xl font-bold text-indigo-900 focus:ring-2 focus:ring-indigo-500 outline-none"  
               /\>  
               \<span className\="absolute right-3 top-3.5 text-slate-400 text-sm"\>₽\</span\>  
             \</div\>  
           \</div\>  
         \</div\>  
       \</div\>

       {/\* 2\. Блок: ОБЪЕМ ПАРТИИ \*/}  
       \<div className\="bg-white p-5 rounded-xl border border-slate-200 shadow-sm"\>  
          \<h3 className\="font-bold text-slate-700 mb-4 flex items-center gap-2"\>  
             \<ArrowRightLeft className\="w-5 h-5 text-slate-500" /\>  
             Объем партии  
           \</h3\>  
           \<div className\="flex flex-col md:flex-row gap-4 items-start md:items-end"\>  
             \<div className\="flex bg-slate-100 p-1 rounded-lg"\>  
               \<button  
                 onClick\={() \=\> setCalcMode('stars')}  
                 className\={\`px-4 py-2 rounded-md text-sm font-medium transition-all ${calcMode \=== 'stars' ? 'bg-white shadow text-slate-800' : 'text-slate-500 hover:text-slate-700'}\`}  
               \>  
                 В Звездах (шт)  
               \</button\>  
               \<button  
                 onClick\={() \=\> setCalcMode('ton')}  
                 className\={\`px-4 py-2 rounded-md text-sm font-medium transition-all ${calcMode \=== 'ton' ? 'bg-white shadow text-slate-800' : 'text-slate-500 hover:text-slate-700'}\`}  
               \>  
                 Бюджет (TON)  
               \</button\>  
             \</div\>

             \<div className\="flex-1 w-full"\>  
               \<label className\="text-xs text-slate-400 block mb-1"\>  
                 {calcMode \=== 'stars' ? 'Сколько звезд продаем?' : 'Сколько TON закупаем?'}  
               \</label\>  
               \<input  
                 type\="number"  
                 value\={inputVolume}  
                 onChange\={(e) \=\> setInputVolume(Number(e.target.value))}  
                 className\="w-full p-2 border border-slate-300 rounded-lg text-lg font-medium focus:ring-2 focus:ring-indigo-500 outline-none"  
               /\>  
             \</div\>  
              
             \<div className\="bg-slate-50 px-4 py-2 rounded-lg border border-slate-100 min-w-\[150px\]"\>  
                \<div className\="text-xs text-slate-400"\>Эквивалент:\</div\>  
                \<div className\="font-mono text-slate-600"\>  
                  {calcMode \=== 'stars'  
                    ? \`≈ ${(inputVolume \* (fragmentBatchCost/fragmentBatchSize)).toFixed(2)} TON\`  
                    : \`≈ ${(inputVolume \* (fragmentBatchSize/fragmentBatchCost)).toFixed(0)} ⭐\`  
                  }  
                \</div\>  
             \</div\>  
           \</div\>  
       \</div\>

       {/\* 3\. Блок: ВЫБОР ИСТОЧНИКА И РАСЧЕТ \*/}  
       \<div\>  
         \<h3 className\="font-bold text-slate-700 mb-3 ml-1"\>Выбор способа закупки\</h3\>  
         \<div className\="space-y-3"\>  
           {calculations.map((item) \=\> {  
             const isSelected \= selectedSourceId \=== item.id;  
             const isProfitable \= item.profitPerStar \> 0;

             return (  
               \<div  
                 key\={item.id}  
                 onClick\={() \=\> setSelectedSourceId(item.id)}  
                 className\={\`relative group cursor-pointer transition-all border rounded-xl p-4 flex flex-col md:flex-row items-center gap-4 ${  
                   isSelected  
                     ? 'bg-indigo-50/50 border-indigo-500 ring-1 ring-indigo-500 shadow-md'  
                     : 'bg-white border-slate-200 hover:border-indigo-300 hover:shadow-sm'  
                 }\`}  
               \>  
                 {/\* Радио-кнопка (Визуально) \*/}  
                 \<div className\={\`shrink-0 ${isSelected ? 'text-indigo-600' : 'text-slate-300'}\`}\>  
                   {isSelected ? \<CheckCircle2 className\="w-6 h-6" /\> : \<Circle className\="w-6 h-6" /\>}  
                 \</div\>

                 {/\* Название и Инпут курса \*/}  
                 \<div className\="flex-1 grid grid-cols-1 md:grid-cols-2 gap-4 w-full"\>  
                   \<div\>  
                     \<div className\="font-bold text-slate-700 text-lg"\>{item.name}\</div\>  
                     \<div className\="text-xs text-slate-400 mb-2"\>{item.desc}\</div\>  
                   \</div\>  
                    
                   \<div onClick\={(e) \=\> e.stopPropagation()}\>  
                     {/\* stopPropagation чтобы клик по инпуту не "мигал" выбором, хотя он и так выбирает \*/}  
                     \<label className\="text-xs text-slate-500 font-semibold block mb-1"\>Курс TON (RUB)\</label\>  
                     \<input  
                       type\="number"  
                       value\={rates\[item.id\]}  
                       onChange\={(e) \=\> handleRateChange(item.id, e.target.value)}  
                       className\={\`w-full p-2 border rounded-lg font-mono text-sm focus:ring-2 focus:ring-indigo-500 outline-none ${isSelected ? 'border-indigo-300 bg-white' : 'border-slate-200 bg-slate-50'}\`}  
                     /\>  
                   \</div\>  
                 \</div\>

                 {/\* Краткий итог по строке \*/}  
                 \<div className\="text-right min-w-\[120px\] md:border-l md:pl-4 md:border-slate-100"\>  
                   \<div className\="text-xs text-slate-400"\>Себест. звезды\</div\>  
                   \<div className\="font-mono text-slate-600"\>{item.costPerStar.toFixed(2)} ₽\</div\>  
                    
                   \<div className\="mt-1"\>  
                     {isProfitable ? (  
                        \<div className\="text-emerald-600 font-bold text-sm flex items-center justify-end gap-1"\>  
                          \<TrendingUp className\="w-3 h-3" /\>  
                          \+{item.profitPerStar.toFixed(2)} ₽  
                        \</div\>  
                     ) : (  
                        \<div className\="text-red-500 font-bold text-sm"\>  
                          {item.profitPerStar.toFixed(2)} ₽  
                        \</div\>  
                     )}  
                   \</div\>  
                 \</div\>  
               \</div\>  
             );  
           })}  
         \</div\>  
       \</div\>

     \</div\>  
   \</div\>  
 );  
};

export default TelegramStarsCalculator;  
