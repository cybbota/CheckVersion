//+------------------------------------------------------------------+
//| БЛОК 1: ШАПКА И ВХОДНЫЕ ПАРАМЕТРЫ                                 |
//| DIZEL_SMT_PRO.mq4                                                |
//| Smart Money Concepts Autonomous Trader                           |
//| Version: 1.0 Final                                               |
//+------------------------------------------------------------------+
#property copyright "DIZEL_SMT_PRO"
#property version   "1.00"
#property description "Полностью автономный SMC советник"
#property description "Мульти-ТФ | CHoCH | BOS | FVG | OB | Sweep | Mitigation"
#property description "Premium/Discount | Killzone | Liquidity Void"
#property strict

#include <stdlib.mqh>

// --- 1. Символ и ТФ ---
extern string   _1_Symbol_Timeframes = "=== 1. Символ и Таймфреймы ===";
extern string   TradeSymbol = "";              // Торгуемый символ (пусто = график)
extern bool     LockSymbol = true;             // Фиксировать символ
extern bool     AdaptiveTimeframes = true;     // Авто-выбор ТФ (true) / Фикс.(false)
extern ENUM_TIMEFRAMES FixedHTF = PERIOD_H4;   // Старший ТФ (если Adaptive=false)
extern ENUM_TIMEFRAMES FixedLTF = PERIOD_M15;  // Младший ТФ (если Adaptive=false)
extern bool     EnableScalping = false;        // Разрешить скальпинг M15/M1

// --- 2. Риск-менеджмент ---
extern string   _2_Risk_Management = "=== 2. Риск-менеджмент ===";
extern double   RiskPercent = 0.5;             // Риск на сделку (%)
extern double   TotalDailyRisk = 1.0;          // Общий дневной риск (%)
extern double   MaxRiskPerPair = 1.0;          // Макс. риск на пару (%)
extern int      MaxConcurrentPairs = 3;        // Макс. пар одновременно
extern double   MinRiskRewardRatio = 2.0;      // Мин. соотношение R/R
extern double   MinSignalScore = 60.0;         // Мин. качество сигнала (0-100)
extern int      CooldownMinutes = 30;          // Кулдаун после закрытия (мин.)

// --- 3. Мульти-цели ---
extern string   _3_Multi_Targets = "=== 3. Мульти-цели (TP) ===";
extern bool     EnableMultiTargets = true;     // Несколько целей
extern double   Target1Volume = 0.50;          // Объём TP1
extern double   Target2Volume = 0.30;          // Объём TP2
extern double   Target3Volume = 0.20;          // Объём TP3

// --- 4. Торговые сессии ---
extern string   _4_Trading_Sessions = "=== 4. Торговые сессии (GMT) ===";
extern bool     EnableSessionFilter = true;    // Фильтр сессий
extern bool     EnableKillzoneOnly = true;     // Только Killzone (08-10, 13-16 GMT)
extern bool     EnableAsianKillzone = false;   // Токийская Killzone (00-02 GMT)
extern bool     EnableAsianSession = false;    // Азия (00-09 GMT)
extern bool     EnableLondonSession = true;    // Лондон (08-17 GMT)
extern bool     EnableNewYorkSession = true;   // Нью-Йорк (13-22 GMT)
extern bool     EnableLondonNYSession = true;  // Лондон+Нью-Йорк

// --- 5. Паттерны ---
extern string   _5_Patterns = "=== 5. Паттерны и анализ ===";
extern bool     EnableFVG = true;              // Fair Value Gap
extern bool     EnableOB = true;               // Order Blocks
extern bool     PreferOB_Over_FVG = true;      // Предпочитать OB
extern bool     UsePremiumDiscountFilter = true; // Фильтр Premium/Discount
extern double   DiscountLevel = 38.2;          // Уровень Discount (%)
extern double   PremiumLevel = 61.8;           // Уровень Premium (%)
extern double   EquilibriumLevel = 50.0;       // Уровень равновесия (%)
extern int      SwingStrength = 5;             // Сила свинга
extern double   EqualTolerancePoints = 5.0;    // Допуск Equal Highs/Lows (пункты)
extern double   MinSwingPullbackATR = 0.3;     // Мин. откат свинга (ATR)

// --- 6. Защита от гэпов ---
extern string   _6_Gap_Protection = "=== 6. Защита от гэпов ===";
extern bool     EnableGapProtection = true;    // Защита от гэпов
extern bool     CloseBeforeWeekend = true;     // Закрывать перед выходными
extern int      WeekendCloseHour = 20;         // Час закрытия в пятницу (GMT)
extern bool     CheckGapBeforeEntry = true;    // Проверять гэп перед входом
extern double   MaxSpreadPoints = 30.0;        // Макс. спред для входа (пункты)
extern int      MaxConnectionLossSeconds = 120; // Макс. потеря связи (сек.)

// --- 7. Инфо-панель ---
extern string   _7_Info_Panel = "=== 7. Инфо-панель ===";
extern int      InfoPanelMode = 2;             // 0=Выкл, 1=Мини, 2=Полная
extern int      InfoPanelCorner = 0;           // 0=Лево-верх, 1=Право-верх

// --- 8. Прочие настройки ---
extern string   _8_Other = "=== 8. Прочие настройки ===";
extern int      Slippage = 30;                 // Проскальзывание
extern int      MagicBase = 202405;            // Базовый Magic Number
extern bool     EnableLogging = true;          // Подробное логирование

//--- 9. Трейлинг-стоп ---
extern string   _9_Trailing_Stop = "=== 9. Трейлинг-стоп ===";
extern bool     EnableTrailingStop = true;      // Включить трейлинг
extern bool     BreakevenAfterTP1 = true;       // Перенос в безубыток после TP1
extern double   BreakevenMinRR = 0.8;           // Минимальный R/R для переноса в БУ

// --- 10. Автономный выбор ТФ под волатильность ---
extern string   _10_Adaptive_TF = "=== 10. Автономный выбор ТФ ===";
extern bool     EnableAdaptiveTFByVolatility = false;  // Включить авто-выбор ТФ
extern double   OptimalVolatilityATRRange = 1.5;       // Оптимальный диапазон ATR (в % от цены)
extern double   MinATRPercent = 0.3;                   // Мин. ATR (% от цены)
extern double   MaxATRPercent = 2.0;                   // Макс. ATR (% от цены)

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 2 🔹                              |
//|    КОНСТАНТЫ, ПЕРЕЧИСЛЕНИЯ, СТРУКТУРЫ, ГЛОБАЛЬНЫЕ ПЕРЕМЕННЫЕ     |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

enum ENUM_MARKET_STRUCTURE
  {
   STRUCTURE_BULLISH,
   STRUCTURE_BEARISH,
   STRUCTURE_UNDEFINED
  };

enum ENUM_PRICE_ZONE
  {
   ZONE_DISCOUNT,
   ZONE_EQUILIBRIUM,
   ZONE_PREMIUM,
   ZONE_UNKNOWN
  };

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                  --- Структуры данных ---                        |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| СТРУКТУРА: Свинговый уровень (экстремум)                         |
//+------------------------------------------------------------------+
struct SwingLevel
  {
   double            price;
   int               bar_index;
   datetime          time;
   bool              is_valid;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Зона ликвидности (BSL/SSL)                            |
//+------------------------------------------------------------------+
struct LiquidityZone
  {
   double            price_top;
   double            price_bottom;
   double            price_center;
   int               swing_count;
   bool              is_active;
   bool              is_bullish;
   datetime          first_time;
   datetime          last_time;
   int               zone_type;        // 0 = BSL (Buy Side Liquidity), 1 = SSL (Sell Side Liquidity)
   bool              is_swept;         // true если зона уже снята (sweep)
   datetime          sweep_time;       // время снятия зоны
   double            sweep_price;      // цена, на которой произошло снятие
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Точка рыночной структуры (BOS/CHoCH)                  |
//+------------------------------------------------------------------+
struct StructurePoint
  {
   double            price;
   int               bar_index;
   datetime          time;
   bool              is_high;
   bool              is_bos;
   bool              is_choch;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Fair Value Gap (FVG)                                  |
//+------------------------------------------------------------------+
struct FVG
  {
   double            price_top;
   double            price_bottom;
   int               bar_index;
   datetime          time;
   bool              is_bullish;
   bool              is_filled;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Order Block (OB)                                      |
//+------------------------------------------------------------------+
struct OrderBlock
  {
   double            price_top;
   double            price_bottom;
   double            open;
   double            close;
   int               bar_index;
   datetime          time;
   bool              is_bullish;
   bool              is_tested;
   bool              is_broken;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Уровень таймфреймов (HTF/LTF)                         |
//+------------------------------------------------------------------+
struct TFLevel
  {
   ENUM_TIMEFRAMES   htf;
   ENUM_TIMEFRAMES   ltf;
   string            name;
   int               weight;
   bool              enabled;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Оценка качества сигнала                               |
//+------------------------------------------------------------------+
struct SignalQuality
  {
   TFLevel           level;
   double            score;
   double            structure_score;
   double            liquidity_score;
   double            pattern_score;
   double            volatility_score;
   bool              is_buy;
   bool              is_valid;
   string            rejection_reason;
   double            entry_price;
   double            stop_loss;
   double            take_profit;
   double            risk_reward;
   datetime          signal_time;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Уровни сделки (SL/TP/RR)                              |
//+------------------------------------------------------------------+
struct TradeLevels
  {
   double            entry_price;
   double            stop_loss;
   double            take_profit;
   double            sl_points;
   double            tp_points;
   double            risk_reward;
   bool              is_valid;
   string            rejection_reason;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Цель для мульти-тейкпрофита                           |
//+------------------------------------------------------------------+
struct LiquidityTarget
  {
   double            price;
   double            volume_percent;
   bool              is_valid;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Группа ордеров (мульти-цели)                          |
//+------------------------------------------------------------------+
struct TradeGroup
  {
   int               tickets[5];
   int               order_count;
   double            total_volume;
   double            stop_loss;
   double            entry_price;
   bool              is_buy;
   datetime          open_time;
   string            group_name;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Контекст ордера (для восстановления)                  |
//+------------------------------------------------------------------+
struct OrderContext
  {
   int               ticket;
   bool              is_buy;
   double            entry_price;
   double            stop_loss;
   double            take_profit;
   double            lot_size;
   string            comment;
   datetime          open_time;
   int               target_index;
   bool              is_active;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Трекер закрытия свечей                                |
//+------------------------------------------------------------------+
struct BarTracker
  {
   ENUM_TIMEFRAMES   tf;
   datetime          last_bar_time;
   string            description;
   bool              is_ltf;
   bool              is_htf;
  };

//+------------------------------------------------------------------+
//| СТРУКТУРА: Торговая сессия                                       |
//+------------------------------------------------------------------+
struct TradingSession
  {
   string            name;
   int               start_hour;
   int               start_minute;
   int               end_hour;
   int               end_minute;
   bool              enabled;
  };

//
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                 --- Глобальные переменные ---                    |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
string            g_symbol;
bool              g_trackers_initialized = false;
datetime          g_last_close_time = 0;
datetime          g_last_tick_received = 0;
datetime          g_last_heartbeat = 0;
int               g_user_period = 0;

TFLevel           g_tf_levels[];
BarTracker        g_bar_trackers[];
TradingSession    g_sessions[];

StructurePoint    g_htf_structure[];

LiquidityZone     g_bsl_zones[];
LiquidityZone     g_ssl_zones[];

FVG               g_ltf_fvgs[];
OrderBlock        g_ltf_obs[];

SignalQuality     g_pending_signals[10];
int               g_pending_signal_count = 0;

OrderContext      g_active_orders[];
int               g_active_order_count = 0;

LiquidityZone     g_last_swept_zone;
FVG               g_last_active_fvg;
OrderBlock        g_last_active_ob;
TradeLevels       g_last_trade_levels;
bool              g_has_last_signal = false;

bool              signal_buy = false;
bool              signal_sell = false;
double            current_price = 0;
double            current_spread = 0;

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 3 🔹                              |
//|                        ИНИЦИАЛИЗАЦИЯ                             |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Инициализация уровней таймфреймов (адаптивный режим)             |
//+------------------------------------------------------------------+
void InitializeTFLevels()
  {
   ArrayResize(g_tf_levels, 4);

   g_tf_levels[0].htf = PERIOD_D1;
   g_tf_levels[0].ltf = PERIOD_H4;
   g_tf_levels[0].name = "D1/H4";
   g_tf_levels[0].weight = 4;
   g_tf_levels[0].enabled = true;

   g_tf_levels[1].htf = PERIOD_H4;
   g_tf_levels[1].ltf = PERIOD_M15;
   g_tf_levels[1].name = "H4/M15";
   g_tf_levels[1].weight = 3;
   g_tf_levels[1].enabled = true;

   g_tf_levels[2].htf = PERIOD_H1;
   g_tf_levels[2].ltf = PERIOD_M5;
   g_tf_levels[2].name = "H1/M5";
   g_tf_levels[2].weight = 2;
   g_tf_levels[2].enabled = true;

   g_tf_levels[3].htf = PERIOD_M15;
   g_tf_levels[3].ltf = PERIOD_M1;
   g_tf_levels[3].name = "M15/M1 (Скальпинг)";
   g_tf_levels[3].weight = 1;
   g_tf_levels[3].enabled = EnableScalping;
  }

//+------------------------------------------------------------------+
//| Инициализация торговых сессий                                     |
//+------------------------------------------------------------------+
void InitializeSessions()
  {
   ArrayResize(g_sessions, 4);

// Asian сессия (Токио, Сидней, Гонконг)
   g_sessions[0].name = "Asian";
   g_sessions[0].start_hour = 0;
   g_sessions[0].start_minute = 0;
   g_sessions[0].end_hour = 9;
   g_sessions[0].end_minute = 0;
   g_sessions[0].enabled = EnableAsianSession;

// London сессия
   g_sessions[1].name = "London";
   g_sessions[1].start_hour = 8;
   g_sessions[1].start_minute = 0;
   g_sessions[1].end_hour = 17;
   g_sessions[1].end_minute = 0;
   g_sessions[1].enabled = EnableLondonSession;

// New York сессия
   g_sessions[2].name = "NewYork";
   g_sessions[2].start_hour = 13;
   g_sessions[2].start_minute = 0;
   g_sessions[2].end_hour = 22;
   g_sessions[2].end_minute = 0;
   g_sessions[2].enabled = EnableNewYorkSession;

// London+NY комбинированная (пересечение)
   g_sessions[3].name = "London+NY";
   g_sessions[3].start_hour = 13;
   g_sessions[3].start_minute = 0;
   g_sessions[3].end_hour = 17;
   g_sessions[3].end_minute = 0;
   g_sessions[3].enabled = EnableLondonNYSession;
  }

//+------------------------------------------------------------------+
//| Инициализация трекеров свечей для всех ТФ                        |
//+------------------------------------------------------------------+
void InitializeBarTrackers()
  {
   ArrayResize(g_bar_trackers, 0);

   ENUM_TIMEFRAMES all_tf[];
   ArrayResize(all_tf, 0);

   for(int i = 0; i < ArraySize(g_tf_levels); i++)
     {
      if(!g_tf_levels[i].enabled)
         continue;
      bool htf_exists = false, ltf_exists = false;
      for(int j = 0; j < ArraySize(all_tf); j++)
        {
         if(all_tf[j] == g_tf_levels[i].htf)
            htf_exists = true;
         if(all_tf[j] == g_tf_levels[i].ltf)
            ltf_exists = true;
        }
      if(!htf_exists)
        {
         int s = ArraySize(all_tf);
         ArrayResize(all_tf, s + 1);
         all_tf[s] = g_tf_levels[i].htf;
        }
      if(!ltf_exists)
        {
         int s = ArraySize(all_tf);
         ArrayResize(all_tf, s + 1);
         all_tf[s] = g_tf_levels[i].ltf;
        }
     }

   if(!AdaptiveTimeframes)
     {
      bool htf_ex = false, ltf_ex = false;
      for(int j = 0; j < ArraySize(all_tf); j++)
        {
         if(all_tf[j] == FixedHTF)
            htf_ex = true;
         if(all_tf[j] == FixedLTF)
            ltf_ex = true;
        }
      if(!htf_ex)
        {
         int s = ArraySize(all_tf);
         ArrayResize(all_tf, s + 1);
         all_tf[s] = FixedHTF;
        }
      if(!ltf_ex)
        {
         int s = ArraySize(all_tf);
         ArrayResize(all_tf, s + 1);
         all_tf[s] = FixedLTF;
        }
     }

   for(int i = 0; i < ArraySize(all_tf) - 1; i++)
     {
      for(int j = i + 1; j < ArraySize(all_tf); j++)
        {
         if(all_tf[i] > all_tf[j])
           {
            ENUM_TIMEFRAMES tmp = all_tf[i];
            all_tf[i] = all_tf[j];
            all_tf[j] = tmp;
           }
        }
     }

   for(int i = 0; i < ArraySize(all_tf); i++)
     {
      int size = ArraySize(g_bar_trackers);
      ArrayResize(g_bar_trackers, size + 1);
      g_bar_trackers[size].tf = all_tf[i];
      g_bar_trackers[size].last_bar_time = 0;
      g_bar_trackers[size].description = EnumToString(all_tf[i]);
      g_bar_trackers[size].is_ltf = false;
      g_bar_trackers[size].is_htf = false;
     }

   for(int i = 0; i < ArraySize(g_tf_levels); i++)
     {
      if(!g_tf_levels[i].enabled)
         continue;
      for(int j = 0; j < ArraySize(g_bar_trackers); j++)
        {
         if(g_bar_trackers[j].tf == g_tf_levels[i].ltf)
            g_bar_trackers[j].is_ltf = true;
         if(g_bar_trackers[j].tf == g_tf_levels[i].htf)
            g_bar_trackers[j].is_htf = true;
        }
     }

   if(!AdaptiveTimeframes)
     {
      for(int j = 0; j < ArraySize(g_bar_trackers); j++)
        {
         if(g_bar_trackers[j].tf == FixedLTF)
            g_bar_trackers[j].is_ltf = true;
         if(g_bar_trackers[j].tf == FixedHTF)
            g_bar_trackers[j].is_htf = true;
        }
     }

   g_trackers_initialized = true;
  }

//+------------------------------------------------------------------+
//| Инициализация символа и проверка таймфреймов                     |
//+------------------------------------------------------------------+
bool InitializeSymbolAndTimeframes()
  {
   if(StringLen(TradeSymbol) > 0)
     {
      g_symbol = TradeSymbol;
      StringToUpper(g_symbol);
     }
   else
     {
      g_symbol = Symbol();
     }

   if(SymbolInfoDouble(g_symbol, SYMBOL_BID) <= 0)
     {
      Print("ОШИБКА: Символ ", g_symbol, " не найден!");
      return false;
     }

   if(!AdaptiveTimeframes && FixedLTF >= FixedHTF)
     {
      Print("ОШИБКА: LTF должен быть МЕНЬШЕ HTF!");
      return false;
     }

   g_user_period = Period();
   return true;
  }

//+------------------------------------------------------------------+
//| Загрузка исторических данных для всех ТФ                         |
//+------------------------------------------------------------------+
void LoadHistoricalData()
  {
   for(int i = 0; i < ArraySize(g_bar_trackers); i++)
     {
      int loaded = iBars(g_symbol, g_bar_trackers[i].tf);
      if(loaded < 100)
        {
         Print("⚠ МАЛО ДАННЫХ для ", g_symbol, " ", g_bar_trackers[i].description);
        }
     }
  }

//+------------------------------------------------------------------+
//|                         ИНИЦИАЛИЗАЦИЯ                            |
//+------------------------------------------------------------------+
int OnInit()
  {
   if(!InitializeSymbolAndTimeframes())
      return INIT_PARAMETERS_INCORRECT;

// Инициализация глобальных переменных
   g_last_tick_received = TimeCurrent(); // ФИКС: чтобы не было ложной потери связи
   g_last_heartbeat = TimeCurrent();
   for(int i = 0; i < 10; i++)
     {
      // Очистка pending signals
      g_pending_signals[i].score = 0;
      g_pending_signals[i].is_valid = false;
      g_pending_signals[i].rejection_reason = "";

      // Очистка active orders
      g_active_orders[i].ticket = -1;
      g_active_orders[i].is_active = false;
     }

// Проверка котировок
   if(GetCurrentBid() <= 0 || GetCurrentAsk() <= 0)
     {
      Print("ОШИБКА: Нет котировок по символу ", g_symbol);
      return INIT_FAILED;
     }

// Применяем настройки в зависимости от символа
   ApplySymbolSpecificSettings(g_symbol);

   if(RiskPercent <= 0 || RiskPercent > 10)
     {
      Print("ОШИБКА: RiskPercent 0.01-10");
      return INIT_PARAMETERS_INCORRECT;
     }
   if(MinRiskRewardRatio < 1.0)
     {
      Print("ОШИБКА: MinRiskRewardRatio >= 1.0");
      return INIT_PARAMETERS_INCORRECT;
     }
   if(SwingStrength < 2 || SwingStrength > 20)
     {
      Print("ОШИБКА: SwingStrength 2-20");
      return INIT_PARAMETERS_INCORRECT;
     }

   InitializeTFLevels();
   InitializeBarTrackers();
   LoadHistoricalData();

// Автономный выбор ТФ под волатильность (если включён)
   if(EnableAdaptiveTFByVolatility)
     {
      ApplyAdaptiveTimeframes();
      // Переинициализируем трекеры с новыми ТФ
      InitializeBarTrackers();
      // Данные уже загружены, повторно не загружаем
     }

   InitializeSessions();
   g_sessions[0].enabled = EnableAsianSession;
   g_sessions[1].enabled = EnableLondonSession;
   g_sessions[2].enabled = EnableNewYorkSession;

   g_last_close_time = 0;
   RecoverAfterRestart();

   EventSetTimer(30);
   Log("═══════════════════════════════════════");
   Log("  DIZEL_SMT_PRO v1.0 успешно запущен");
   Log("═══════════════════════════════════════");
   return INIT_SUCCEEDED;
  }

//+------------------------------------------------------------------+
//|                        ДЕИНИЦИАЛИЗАЦИЯ                           |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
   EventKillTimer();
   if(g_active_order_count > 0)
      SaveOrderContextToFile(g_active_orders, g_active_order_count);
   ClearAllPanelObjects();
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 4 🔹                              |
//|                    ВСПОМОГАТЕЛЬНЫЕ ФУНКЦИИ                       |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Нормализация цены (округление до digits)                         |
//+------------------------------------------------------------------+
double NormalizePrice(double price)
  {
   return NormalizeDouble(price, GetDigits());
  }

//+------------------------------------------------------------------+
//| Получение High бара на указанном ТФ                              |
//+------------------------------------------------------------------+
double GetHigh(ENUM_TIMEFRAMES tf, int shift) { return iHigh(g_symbol, tf, shift); }

//+------------------------------------------------------------------+
//| Получение Low бара на указанном ТФ                               |
//+------------------------------------------------------------------+
double GetLow(ENUM_TIMEFRAMES tf, int shift) { return iLow(g_symbol, tf, shift); }

//+------------------------------------------------------------------+
//| Получение Close бара на указанном ТФ                             |
//+------------------------------------------------------------------+
double GetClose(ENUM_TIMEFRAMES tf, int shift) { return iClose(g_symbol, tf, shift); }

//+------------------------------------------------------------------+
//| Получение Open бара на указанном ТФ                              |
//+------------------------------------------------------------------+
double GetOpen(ENUM_TIMEFRAMES tf, int shift) { return iOpen(g_symbol, tf, shift); }
//+------------------------------------------------------------------+
//| Получение времени бара на указанном ТФ                           |
//+------------------------------------------------------------------+
datetime GetTime(ENUM_TIMEFRAMES tf, int shift) { return iTime(g_symbol, tf, shift); }

//+------------------------------------------------------------------+
//| Получение количества баров на указанном ТФ                       |
//+------------------------------------------------------------------+
int GetBars(ENUM_TIMEFRAMES tf) { return iBars(g_symbol, tf); }

//+------------------------------------------------------------------+
//| Получение значения ATR на указанном ТФ                           |
//+------------------------------------------------------------------+
double GetATR(ENUM_TIMEFRAMES tf, int period = 14, int shift = 0) { return iATR(g_symbol, tf, period, shift); }

//+------------------------------------------------------------------+
//| Получение текущей цены Bid                                       |
//+------------------------------------------------------------------+
double GetCurrentBid() { return SymbolInfoDouble(g_symbol, SYMBOL_BID); }

//+------------------------------------------------------------------+
//| Получение текущей цены Ask                                       |
//+------------------------------------------------------------------+
double GetCurrentAsk() { return SymbolInfoDouble(g_symbol, SYMBOL_ASK); }

//+------------------------------------------------------------------+
//| Получение текущего спреда                                        |
//+------------------------------------------------------------------+
double GetCurrentSpread() { return GetCurrentAsk() - GetCurrentBid(); }

//+------------------------------------------------------------------+
//| Получение количества знаков после запятой                        |
//+------------------------------------------------------------------+
int GetDigits() { return (int)SymbolInfoInteger(g_symbol, SYMBOL_DIGITS); }

//+------------------------------------------------------------------+
//| Получение размера пункта для символа                             |
//+------------------------------------------------------------------+
double GetPoint() { return SymbolInfoDouble(g_symbol, SYMBOL_POINT); }

//+------------------------------------------------------------------+
//| Получение текущего времени GMT                                   |
//+------------------------------------------------------------------+
datetime GetGMTTime() { return TimeGMT(); }

//+------------------------------------------------------------------+
//| Логирование с префиксом [DIZEL_SMT]                              |
//+------------------------------------------------------------------+
void Log(string message) { if(EnableLogging) Print("[DIZEL_SMT] ", message); }

//+------------------------------------------------------------------+
//| Получение количества секунд в периоде (пользовательская)         |
//+------------------------------------------------------------------+
int GetPeriodSeconds(ENUM_TIMEFRAMES tf)
  {
   switch(tf)
     {
      case PERIOD_M1:
         return 60;
      case PERIOD_M5:
         return 300;
      case PERIOD_M15:
         return 900;
      case PERIOD_M30:
         return 1800;
      case PERIOD_H1:
         return 3600;
      case PERIOD_H4:
         return 14400;
      case PERIOD_D1:
         return 86400;
      case PERIOD_W1:
         return 604800;
      case PERIOD_MN1:
         return 2592000;
      default:
         return 3600;
     }
  }

//+------------------------------------------------------------------+
//| Проверка: находится ли текущее серверное время внутри временного окна |
//+------------------------------------------------------------------+
bool IsWithinTimeWindow(int start_hour, int start_min, int end_hour, int end_min)
  {
// Используем серверное время брокера (важно для торговых сессий!)
   datetime now = TimeCurrent();

   MqlDateTime dt;
   TimeToStruct(now, dt);

// Переводим текущее время в минуты от полуночи
   int current_minutes = dt.hour * 60 + dt.min;

// Переводим границы окна в минуты от полуночи
   int start_minutes = start_hour * 60 + start_min;
   int end_minutes = end_hour * 60 + end_min;

// Обработка двух случаев:
// 1. Обычное окно (start <= end) - например, 08:00 - 17:00
// 2. Окно через полночь (start > end) - например, 22:00 - 06:00

   if(start_minutes <= end_minutes)
     {
      // Обычное временное окно (не переходит через полночь)
      return (current_minutes >= start_minutes && current_minutes < end_minutes);
     }
   else
     {
      // Временное окно переходит через полночь (например, 22:00 - 06:00)
      return (current_minutes >= start_minutes || current_minutes < end_minutes);
     }
  }

//+------------------------------------------------------------------+
//| Проверка: закрылась ли новая свеча на указанном ТФ               |
//+------------------------------------------------------------------+
bool IsNewBarClosed(ENUM_TIMEFRAMES tf, datetime &last_known_time)
  {
   datetime current_bar_open = iTime(g_symbol, tf, 0);
   if(current_bar_open != last_known_time && current_bar_open > 0)
     {
      last_known_time = current_bar_open;
      return true;
     }
   return false;
  }

//+------------------------------------------------------------------+
//| Поиск индекса бара по времени (аналог iBarShift)                 |
//+------------------------------------------------------------------+
int iBarShift(string symbol, ENUM_TIMEFRAMES tf, datetime time)
  {
   if(time == 0)
      return -1;
   int total_bars = iBars(symbol, tf);
   for(int i = 0; i < total_bars; i++)
     {
      if(iTime(symbol, tf, i) <= time)
         return i;
     }
   return -1;
  }

//+------------------------------------------------------------------+
//| Генерация Magic Number для ордера                                |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//| Генерация Magic Number (один на весь советник)                    |
//|                                                                   |
//| Magic Number — идентификатор советника. Все ордера этого          |
//| советника имеют одинаковый Magic.                                  |
//|                                                                   |
//| Для разделения ордеров внутри советника используются:             |
//|   • Ticket — уникальный номер ордера (присваивается брокером)    |
//|   • Comment — группировка по целям, таймфреймам, направлению     |
//|                                                                   |
//| Если MagicBase задан во внешних параметрах (>0) — используется он.|
//| Если нет — генерируется автоматически из имени символа.           |
//+------------------------------------------------------------------+
int GetMagicNumber()
  {
// Если MagicBase задан пользователем — используем его
   if(MagicBase > 0)
      return MagicBase;

// Автоматическая генерация уникального Magic из имени символа
// Например: "EURUSD" -> 34851730, "XAUUSD" -> 26548912
   int hash = 0;
   string sym = g_symbol;

   for(int i = 0; i < StringLen(sym); i++)
     {
      hash = ((hash << 5) - hash) + (int)StringGetCharacter(sym, i);
      hash = hash & 0x7FFFFFFF;  // Гарантируем положительное значение
     }

// Диапазон 1 — 2 147 483 647 (максимальный положительный int)
   if(hash < 1)
      hash = 1;

   return hash;
  }

//+------------------------------------------------------------------+
//| Проверка: принадлежит ли ордер нашему советнику                  |
//+------------------------------------------------------------------+
bool IsOurOrder(int ticket)
  {
   if(ticket <= 0)
      return false;

   if(!OrderSelect(ticket, SELECT_BY_TICKET))
      return false;

   int magic = OrderMagicNumber();
   return OrderMagicNumber() == GetMagicNumber();
  }

//+------------------------------------------------------------------+
//| Подсчёт активных экземпляров советника (мульти-парный режим)     |
//+------------------------------------------------------------------+
int GetActiveSMCInstanceCount()
  {
   int count = 0;
   datetime current_time = TimeCurrent();
   for(int i = 0; i < GlobalVariablesTotal(); i++)
     {
      string var_name = GlobalVariableName(i);
      if(StringFind(var_name, "SMC_Active_", 0) == 0)
        {
         if(current_time - (datetime)GlobalVariableGet(var_name) < 300)
            count++;
        }
     }
   return MathMax(1, count);
  }

//+------------------------------------------------------------------+
//| Обновление "сердцебиения" для мульти-парного режима              |
//+------------------------------------------------------------------+
void UpdateHeartbeat()
  {
   if(TimeCurrent() - g_last_heartbeat > 60)
     {
      g_last_heartbeat = TimeCurrent();
      GlobalVariableSet("SMC_Active_" + g_symbol, (double)TimeCurrent());
     }
  }

//+------------------------------------------------------------------+
//| Проверка лимита количества пар в одновременной торговле          |
//+------------------------------------------------------------------+
bool CanTradeThisPair()
  {
   int pairs_in_trade = 0;
   bool this_pair_already_in = false;
   for(int i = 0; i < OrdersTotal(); i++)
     {
      if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES))
        {
         if(StringFind(OrderComment(), "SMC_", 0) == 0)
           {
            if(OrderSymbol() == g_symbol)
               this_pair_already_in = true;
            else
               pairs_in_trade++;
           }
        }
     }
   if(this_pair_already_in)
      return true;
   return pairs_in_trade < MaxConcurrentPairs;
  }

//+------------------------------------------------------------------+
//| Проверка наличия противоположных ордеров                         |
//+------------------------------------------------------------------+
bool HasOppositeOrders(bool want_buy)
  {
   for(int i = 0; i < g_active_order_count; i++)
     {
      if(g_active_orders[i].is_active && g_active_orders[i].is_buy != want_buy)
         return true;
     }
   return false;
  }

//+------------------------------------------------------------------+
//| Подсчёт активных зон ликвидности                                 |
//+------------------------------------------------------------------+
int CountActiveZones(LiquidityZone &zones[])
  {
   int count = 0;
   for(int i = 0; i < ArraySize(zones); i++)
     {
      if(zones[i].is_active)
         count++;
     }
   return count;
  }

//+------------------------------------------------------------------+
//| Подсчёт незаполненных FVG                                        |
//+------------------------------------------------------------------+
int CountUnfilledFVG(FVG &fvgs[])
  {
   int count = 0;
   for(int i = 0; i < ArraySize(fvgs); i++)
     {
      if(!fvgs[i].is_filled)
         count++;
     }
   return count;
  }

//+------------------------------------------------------------------+
//| Подсчёт активных Order Block                                     |
//+------------------------------------------------------------------+
int CountActiveOB(OrderBlock &obs[])
  {
   int count = 0;
   for(int i = 0; i < ArraySize(obs); i++)
     {
      if(!obs[i].is_broken)
         count++;
     }
   return count;
  }

//+------------------------------------------------------------------+
//| Определение рекомендуемой сессии для символа                     |
//+------------------------------------------------------------------+
string GetSymbolSessionConfig(string symbol)
  {
   string upper_symbol = symbol;
   StringToUpper(upper_symbol);

// === МЕТАЛЛЫ ===
   if(StringFind(upper_symbol, "XAU") >= 0)      // Золото
      return "London+NY";
   if(StringFind(upper_symbol, "XAG") >= 0)      // Серебро
      return "London+NY";
   if(StringFind(upper_symbol, "XPT") >= 0)      // Платина
      return "London+NY";
   if(StringFind(upper_symbol, "XPD") >= 0)      // Палладий
      return "London+NY";

// === НЕФТЬ И ГАЗ ===
   if(StringFind(upper_symbol, "WTI") >= 0 ||    // Нефть WTI
      StringFind(upper_symbol, "OIL") >= 0 ||
      StringFind(upper_symbol, "BRENT") >= 0 ||  // Брент
      StringFind(upper_symbol, "BRN") >= 0 ||
      StringFind(upper_symbol, "XBR") >= 0 ||
      StringFind(upper_symbol, "XTI") >= 0 ||
      StringFind(upper_symbol, "NG") >= 0)       // Природный газ
      return "London+NY";

// === ИНДЕКСЫ ===
   if(StringFind(upper_symbol, "SPX") >= 0 ||    // S&P 500
      StringFind(upper_symbol, "SP500") >= 0 ||
      StringFind(upper_symbol, "US500") >= 0 ||
      StringFind(upper_symbol, "NAS") >= 0 ||    // NASDAQ
      StringFind(upper_symbol, "US100") >= 0 ||
      StringFind(upper_symbol, "NDX") >= 0 ||
      StringFind(upper_symbol, "DJI") >= 0 ||    // Dow Jones
      StringFind(upper_symbol, "US30") >= 0 ||
      StringFind(upper_symbol, "DAX") >= 0 ||    // Германия
      StringFind(upper_symbol, "DE40") >= 0 ||
      StringFind(upper_symbol, "FTSE") >= 0 ||   // Великобритания
      StringFind(upper_symbol, "UK100") >= 0 ||
      StringFind(upper_symbol, "CAC") >= 0 ||    // Франция
      StringFind(upper_symbol, "FR40") >= 0 ||
      StringFind(upper_symbol, "NIKKEI") >= 0 || // Япония
      StringFind(upper_symbol, "JP225") >= 0 ||
      StringFind(upper_symbol, "HSI") >= 0 ||    // Гонконг
      StringFind(upper_symbol, "HK50") >= 0)
      return "London+NY";

// === КРИПТОВАЛЮТЫ ===
   if(StringFind(upper_symbol, "BTC") >= 0 ||    // Биткоин
      StringFind(upper_symbol, "ETH") >= 0 ||    // Эфириум
      StringFind(upper_symbol, "LTC") >= 0 ||    // Litecoin
      StringFind(upper_symbol, "XRP") >= 0 ||    // Ripple
      StringFind(upper_symbol, "ADA") >= 0 ||    // Cardano
      StringFind(upper_symbol, "DOT") >= 0 ||    // Polkadot
      StringFind(upper_symbol, "DOGE") >= 0 ||   // Dogecoin
      StringFind(upper_symbol, "SOL") >= 0)      // Solana
      return "London+NY";  // Крипта активна в London+NY из-за волатильности

// === МАЖОРНЫЕ ВАЛЮТНЫЕ ПАРЫ ===
   if(StringFind(upper_symbol, "EUR") >= 0 &&
      (StringFind(upper_symbol, "USD") >= 0 ||   // EURUSD
       StringFind(upper_symbol, "GBP") >= 0 ||   // EURGBP
       StringFind(upper_symbol, "CHF") >= 0))    // EURCHF
      return "London+NY";

   if(StringFind(upper_symbol, "GBP") >= 0 &&
      StringFind(upper_symbol, "USD") >= 0)      // GBPUSD
      return "London";

   if(StringFind(upper_symbol, "USD") >= 0 &&
      StringFind(upper_symbol, "JPY") >= 0)      // USDJPY
      return "Asian+London";

   if(StringFind(upper_symbol, "USD") >= 0 &&
      StringFind(upper_symbol, "CHF") >= 0)      // USDCHF
      return "London+NY";

   if(StringFind(upper_symbol, "USD") >= 0 &&
      StringFind(upper_symbol, "CAD") >= 0)      // USDCAD
      return "London+NY";

   if(StringFind(upper_symbol, "AUD") >= 0 ||
      StringFind(upper_symbol, "NZD") >= 0)
      return "Asian";

// === КРОСС-КУРСЫ ===
   if(StringFind(upper_symbol, "EUR") >= 0 &&
      StringFind(upper_symbol, "JPY") >= 0)      // EURJPY
      return "Asian+London";

   if(StringFind(upper_symbol, "GBP") >= 0 &&
      StringFind(upper_symbol, "JPY") >= 0)      // GBPJPY
      return "Asian+London";

   if(StringFind(upper_symbol, "GBP") >= 0 &&
      StringFind(upper_symbol, "CHF") >= 0)      // GBPCHF
      return "London";

   if(StringFind(upper_symbol, "GBP") >= 0 &&
      StringFind(upper_symbol, "CAD") >= 0)      // GBPCAD
      return "London";

   if(StringFind(upper_symbol, "AUD") >= 0 &&
      StringFind(upper_symbol, "JPY") >= 0)      // AUDJPY
      return "Asian";

   if(StringFind(upper_symbol, "NZD") >= 0 &&
      StringFind(upper_symbol, "JPY") >= 0)      // NZDJPY
      return "Asian";

   if(StringFind(upper_symbol, "AUD") >= 0 &&
      StringFind(upper_symbol, "NZD") >= 0)      // AUDNZD
      return "Asian";

// === СКАНДИНАВСКИЕ ВАЛЮТЫ ===
   if(StringFind(upper_symbol, "SEK") >= 0 ||
      StringFind(upper_symbol, "NOK") >= 0 ||
      StringFind(upper_symbol, "DKK") >= 0)
      return "London";

// === ВОСТОЧНОЕВРОПЕЙСКИЕ ===
   if(StringFind(upper_symbol, "PLN") >= 0 ||
      StringFind(upper_symbol, "CZK") >= 0 ||
      StringFind(upper_symbol, "HUF") >= 0)
      return "London";

// === АФРИКАНСКИЕ ===
   if(StringFind(upper_symbol, "ZAR") >= 0)      // USDZAR, EURZAR
      return "London";

// === ЭКЗОТИЧЕСКИЕ ===
   if(StringFind(upper_symbol, "TRY") >= 0 ||    // Турецкая лира
      StringFind(upper_symbol, "MXN") >= 0 ||    // Мексиканское песо
      StringFind(upper_symbol, "BRL") >= 0 ||    // Бразильский реал
      StringFind(upper_symbol, "RUB") >= 0 ||    // Российский рубль
      StringFind(upper_symbol, "CNH") >= 0 ||    // Китайский юань offshore
      StringFind(upper_symbol, "SGD") >= 0 ||    // Сингапурский доллар
      StringFind(upper_symbol, "HKD") >= 0 ||    // Гонконгский доллар
      StringFind(upper_symbol, "KRW") >= 0)      // Южнокорейская вона
      return "London+NY";

   return "Default";
  }

//+------------------------------------------------------------------+
//| Применение настроек в зависимости от символа                     |
//+------------------------------------------------------------------+
void ApplySymbolSpecificSettings(string symbol)
  {
   string upper = symbol;
   StringToUpper(upper);

   Log("Применяем автоматические настройки для " + symbol);

// === МЕТАЛЛЫ ===
   if(StringFind(upper, "XAU") >= 0 || StringFind(upper, "GOLD") >= 0)
     {
      RiskPercent = 0.5;
      MinRiskRewardRatio = 2.0;
      MinSignalScore = 55;
      MaxSpreadPoints = 30;
      Slippage = 50;
      Log("→ Настройки для ЗОЛОТА (XAU)");
     }
   else
      if(StringFind(upper, "XAG") >= 0 || StringFind(upper, "SILVER") >= 0)
        {
         RiskPercent = 0.7;
         MinRiskRewardRatio = 2.5;
         MinSignalScore = 50;
         MaxSpreadPoints = 40;
         Slippage = 50;
         Log("→ Настройки для СЕРЕБРА (XAG)");
        }
      // === НЕФТЬ ===
      else
         if(StringFind(upper, "WTI") >= 0 || StringFind(upper, "BRENT") >= 0 ||
            StringFind(upper, "OIL") >= 0 || StringFind(upper, "XBR") >= 0 ||
            StringFind(upper, "XTI") >= 0)
           {
            RiskPercent = 0.7;
            MinRiskRewardRatio = 2.5;
            MinSignalScore = 60;
            MaxSpreadPoints = 20;
            Slippage = 30;
            CooldownMinutes = 60;
            Log("→ Настройки для НЕФТИ");
           }
         // === КРИПТОВАЛЮТЫ ===
         else
            if(StringFind(upper, "BTC") >= 0 || StringFind(upper, "ETH") >= 0)
              {
               RiskPercent = 0.5;
               MinRiskRewardRatio = 2.5;
               MinSignalScore = 65;
               MaxSpreadPoints = 50;
               Slippage = 100;
               EnableSessionFilter = false;
               Log("→ Настройки для КРИПТОВАЛЮТ");
              }
            // === ИНДЕКСЫ ===
            else
               if(StringFind(upper, "SPX") >= 0 || StringFind(upper, "SP500") >= 0 ||
                  StringFind(upper, "NAS") >= 0 || StringFind(upper, "US100") >= 0 ||
                  StringFind(upper, "DJI") >= 0 || StringFind(upper, "US30") >= 0 ||
                  StringFind(upper, "DAX") >= 0 || StringFind(upper, "DE40") >= 0 ||
                  StringFind(upper, "FTSE") >= 0 || StringFind(upper, "UK100") >= 0)
                 {
                  RiskPercent = 0.5;
                  MinRiskRewardRatio = 2.0;
                  MinSignalScore = 55;
                  MaxSpreadPoints = 20;
                  Slippage = 50;
                  Log("→ Настройки для ИНДЕКСОВ");
                 }
               // === EURUSD ===
               else
                  if(StringFind(upper, "EURUSD") >= 0)
                    {
                     RiskPercent = 1.0;
                     MinSignalScore = 50;
                     Log("→ Настройки для EURUSD");
                    }
                  // === GBPUSD ===
                  else
                     if(StringFind(upper, "GBPUSD") >= 0)
                       {
                        RiskPercent = 0.8;
                        MinSignalScore = 55;
                        Log("→ Настройки для GBPUSD");
                       }
                     // === USDJPY ===
                     else
                        if(StringFind(upper, "USDJPY") >= 0)
                          {
                           RiskPercent = 0.8;
                           MinSignalScore = 55;
                           MaxSpreadPoints = 15;
                           Log("→ Настройки для USDJPY");
                          }
                        // === AUDUSD / NZDUSD ===
                        else
                           if(StringFind(upper, "AUDUSD") >= 0 || StringFind(upper, "NZDUSD") >= 0)
                             {
                              RiskPercent = 0.8;
                              MinSignalScore = 60;
                              Log("→ Настройки для " + (StringFind(upper, "AUD") >= 0 ? "AUDUSD" : "NZDUSD"));
                             }
                           // === КРОСС-КУРСЫ С JPY ===
                           else
                              if((StringFind(upper, "JPY") >= 0) &&
                                 (StringFind(upper, "EUR") >= 0 || StringFind(upper, "GBP") >= 0 ||
                                  StringFind(upper, "CAD") >= 0 || StringFind(upper, "CHF") >= 0))
                                {
                                 RiskPercent = 0.5;
                                 MinRiskRewardRatio = 2.5;
                                 MinSignalScore = 60;
                                 Log("→ Настройки для JPY-кроссов");
                                }
                              // === ПО УМОЛЧАНИЮ ===
                              else
                                {
                                 Log("→ Используются стандартные настройки");
                                }

   Log("  RiskPercent=" + DoubleToString(RiskPercent, 1) +
       "% | MinRR=" + DoubleToString(MinRiskRewardRatio, 1) +
       " | MinScore=" + DoubleToString(MinSignalScore, 0));
  }

//+------------------------------------------------------------------+
//| Получение ATR в процентах от текущей цены для указанного ТФ      |
//+------------------------------------------------------------------+
double GetATRPercent(ENUM_TIMEFRAMES tf)
  {
   double atr = GetATR(tf, 14, 0);
   double price = GetCurrentBid();
   if(price <= 0.0001) // Защита
      return 0;
   return (atr / price) * 100.0;
  }

//+------------------------------------------------------------------+
//| Автономный выбор оптимального ТФ на основе волатильности        |
//| Возвращает: ENUM_TIMEFRAMES (лучший ТФ для торговли)             |
//+------------------------------------------------------------------+
ENUM_TIMEFRAMES SelectOptimalTimeframeByVolatility()
  {
   if(!EnableAdaptiveTFByVolatility)
      return PERIOD_CURRENT;  // Используем текущий ТФ графика

// Список ТФ для анализа (от меньшего к большему)
   ENUM_TIMEFRAMES tf_list[] = {PERIOD_M5, PERIOD_M15, PERIOD_H1, PERIOD_H4, PERIOD_D1};
   string tf_names[] = {"M5", "M15", "H1", "H4", "D1"};

   double best_score = -1;
   ENUM_TIMEFRAMES best_tf = PERIOD_H1;  // По умолчанию H1
   string best_name = "H1";

   for(int i = 0; i < ArraySize(tf_list); i++)
     {
      double atr_percent = GetATRPercent(tf_list[i]);
      if(atr_percent <= 0)
         continue;

      // Оценка ТФ:
      // Чем ближе ATR% к OptimalVolatilityATRRange, тем выше оценка
      double score = 0;

      // 1. Попадание в оптимальный диапазон (основной фактор)
      if(atr_percent >= MinATRPercent && atr_percent <= MaxATRPercent)
        {
         // Чем ближе к OptimalVolatilityATRRange, тем выше оценка
         double diff = MathAbs(atr_percent - OptimalVolatilityATRRange);
         score = 100.0 * (1.0 - MathMin(1.0, diff / OptimalVolatilityATRRange));
        }
      else
        {
         // Вне диапазона — штраф
         score = 10.0;
        }

      // 2. Бонус за более высокий ТФ (для надёжности структуры)
      if(tf_list[i] >= PERIOD_H4)
         score += 15;
      else
         if(tf_list[i] >= PERIOD_H1)
            score += 10;
         else
            if(tf_list[i] >= PERIOD_M15)
               score += 5;

      // 3. Бонус за стабильность (чем меньше ATR, тем стабильнее, но не ниже MinATRPercent)
      if(atr_percent > MinATRPercent && atr_percent < OptimalVolatilityATRRange)
         score += 5;

      Log("[AdaptiveTF] " + tf_names[i] + ": ATR=" + DoubleToString(atr_percent, 2) +
          "% | Score=" + DoubleToString(score, 1));

      if(score > best_score)
        {
         best_score = score;
         best_tf = tf_list[i];
         best_name = tf_names[i];
        }
     }

   if(best_score > 0)
     {
      Log("[AdaptiveTF] ✅ Выбран оптимальный ТФ: " + best_name +
          " (Score=" + DoubleToString(best_score, 1) + ")");
     }
   else
     {
      Log("[AdaptiveTF] ⚠ Не удалось определить ТФ, используем H1");
      best_tf = PERIOD_H1;
     }

   return best_tf;
  }

//+------------------------------------------------------------------+
//| Применение автономного ТФ к настройкам адаптивного режима        |
//+------------------------------------------------------------------+
void ApplyAdaptiveTimeframes()
  {
   if(!EnableAdaptiveTFByVolatility)
      return;

   ENUM_TIMEFRAMES optimal_tf = SelectOptimalTimeframeByVolatility();

   switch(optimal_tf)
     {
      case PERIOD_M5:
         if(AdaptiveTimeframes)
           {
            for(int i = 0; i < ArraySize(g_tf_levels); i++)
              {
               if(g_tf_levels[i].name == "H1/M5")
                 {
                  g_tf_levels[i].enabled = true;
                  g_tf_levels[i].weight = 3;
                 }
               else
                  if(g_tf_levels[i].name == "H4/M15")
                    {
                     g_tf_levels[i].enabled = false;
                    }
              }
           }
         else
           {
            FixedHTF = PERIOD_H1;
            FixedLTF = PERIOD_M1;
           }
         Log("[AdaptiveTF] Установлен режим: HTF=H1, LTF=M1 (для скальпинга)");
         break;

      case PERIOD_M15:
         if(AdaptiveTimeframes)
           {
            for(int i = 0; i < ArraySize(g_tf_levels); i++)
              {
               if(g_tf_levels[i].name == "H4/M15")
                 {
                  g_tf_levels[i].enabled = true;
                  g_tf_levels[i].weight = 3;
                 }
               else
                  if(g_tf_levels[i].name == "H1/M5")
                    {
                     g_tf_levels[i].enabled = EnableScalping;
                    }
              }
           }
         else
           {
            FixedHTF = PERIOD_H4;
            FixedLTF = PERIOD_M5;
           }
         Log("[AdaptiveTF] Установлен режим: HTF=H4, LTF=M5 (стандартный)");
         break;

      case PERIOD_H1:
         if(AdaptiveTimeframes)
           {
            for(int i = 0; i < ArraySize(g_tf_levels); i++)
              {
               if(g_tf_levels[i].name == "D1/H4")
                 {
                  g_tf_levels[i].enabled = true;
                  g_tf_levels[i].weight = 4;
                 }
              }
           }
         else
           {
            FixedHTF = PERIOD_D1;
            FixedLTF = PERIOD_M15;
           }
         Log("[AdaptiveTF] Установлен режим: HTF=D1, LTF=M15 (среднесрочный)");
         break;

      case PERIOD_H4:
         if(AdaptiveTimeframes)
           {
            for(int i = 0; i < ArraySize(g_tf_levels); i++)
              {
               if(g_tf_levels[i].name == "D1/H4")
                 {
                  g_tf_levels[i].enabled = true;
                  g_tf_levels[i].weight = 4;
                 }
              }
           }
         else
           {
            FixedHTF = PERIOD_D1;
            FixedLTF = PERIOD_H1;
           }
         Log("[AdaptiveTF] Установлен режим: HTF=D1, LTF=H1 (долгосрочный)");
         break;

      case PERIOD_D1:
         if(AdaptiveTimeframes)
           {
            // Проверяем, существует ли уже уровень W1/H4
            bool w1_exists = false;
            for(int i = 0; i < ArraySize(g_tf_levels); i++)
              {
               if(g_tf_levels[i].name == "W1/H4")
                 {
                  w1_exists = true;
                  g_tf_levels[i].enabled = true;
                  g_tf_levels[i].weight = 5;
                  break;
                 }
              }

            // Добавляем только если не существует
            if(!w1_exists)
              {
               int size = ArraySize(g_tf_levels);
               ArrayResize(g_tf_levels, size + 1);
               g_tf_levels[size].htf = PERIOD_W1;
               g_tf_levels[size].ltf = PERIOD_H4;
               g_tf_levels[size].name = "W1/H4";
               g_tf_levels[size].weight = 5;
               g_tf_levels[size].enabled = true;
              }
           }
         else
           {
            FixedHTF = PERIOD_W1;
            FixedLTF = PERIOD_H4;
           }
         Log("[AdaptiveTF] Установлен режим: HTF=W1, LTF=H4 (инвестиционный)");
         break;

      default:
         Log("[AdaptiveTF] Используются настройки по умолчанию");
         break;
     }
  }

//+------------------------------------------------------------------+
//| Проверка связи (получаем ли тики)                                |
//+------------------------------------------------------------------+
bool IsConnectionAlive()
  {
   if(g_last_tick_received == 0)
     {
      g_last_tick_received = TimeCurrent();
      return true;
     }
   int seconds_since = (int)(TimeCurrent() - g_last_tick_received);
   if(seconds_since > MaxConnectionLossSeconds)
     {
      Log("🚨 ПОТЕРЯ СВЯЗИ! " + IntegerToString(seconds_since) + " сек.");
      return false;
     }
   return true;
  }

//+------------------------------------------------------------------+
//| Адаптивный трейлинг-стоп с учётом ликвидности                    |
//+------------------------------------------------------------------+
void UpdateTrailingStopForGroup(TradeGroup &group, ENUM_TIMEFRAMES tf)
  {
   if(!EnableTrailingStop)
      return;
   if(group.order_count == 0)
      return;
   if(group.stop_loss <= 0)
      return;

   double point = GetPoint();
   double atr = GetATR(tf, 14, 0);
   if(atr <= 0)
      atr = point * 100;

   double spread = GetCurrentSpread();
   double spread_pips = spread / point;
   double price_now = group.is_buy ? GetCurrentBid() : GetCurrentAsk();
   double entry_price = group.entry_price;

// === 1. Расчёт текущей прибыли ===
   double profit_pips = 0;
   if(group.is_buy)
      profit_pips = (price_now - entry_price) / point;
   else
      profit_pips = (entry_price - price_now) / point;

   double profit_atr = profit_pips * point / atr;
   double sl_distance = MathAbs(group.stop_loss - entry_price) / point;
   double current_rr = (sl_distance > 0) ? profit_pips / sl_distance : 0;

// === 2. Поиск ближайшей значимой зоны ===
   double nearest_zone_distance = 99999;
   double nearest_zone_price = 0;
   string zone_type = "";

   if(group.is_buy)
     {
      // Для BUY ищем ближайшую SSL (сопротивление) как цель
      for(int i = 0; i < ArraySize(g_ssl_zones); i++)
        {
         if(g_ssl_zones[i].is_active && g_ssl_zones[i].price_center > price_now)
           {
            double dist = (g_ssl_zones[i].price_center - price_now) / point;
            if(dist > 0 && dist < nearest_zone_distance)
              {
               nearest_zone_distance = dist;
               nearest_zone_price = g_ssl_zones[i].price_center;
               zone_type = "SSL";
              }
           }
        }
     }
   else
     {
      // Для SELL ищем ближайшую BSL (поддержку)
      for(int i = 0; i < ArraySize(g_bsl_zones); i++)
        {
         if(g_bsl_zones[i].is_active && g_bsl_zones[i].price_center < price_now)
           {
            double dist = (price_now - g_bsl_zones[i].price_center) / point;
            if(dist > 0 && dist < nearest_zone_distance)
              {
               nearest_zone_distance = dist;
               nearest_zone_price = g_bsl_zones[i].price_center;
               zone_type = "BSL";
              }
           }
        }
     }

// === 3. Расчёт базовой дистанции трейлинга ===
   double trailing_distance = 0;
   double activation_profit = 0;

// 3.1. База от ATR (0.8-1.5 ATR в зависимости от волатильности)
   double atr_ratio = atr / (point * 100);  // Нормализуем ATR
   if(atr_ratio > 2.0)  // Высокая волатильность
     {
      trailing_distance = atr * 1.5;
      activation_profit = atr * 2.0;
     }
   else
      if(atr_ratio > 1.0)  // Средняя волатильность
        {
         trailing_distance = atr * 1.0;
         activation_profit = atr * 1.5;
        }
      else  // Низкая волатильность
        {
         trailing_distance = atr * 0.8;
         activation_profit = atr * 1.2;
        }

// 3.2. Корректировка на спред (чем шире спред, тем больше дистанция)
   if(spread_pips > 15)
      trailing_distance += spread / 2;

// 3.3. Корректировка на ближайшую зону
   bool has_nearby_zone = (nearest_zone_distance < 100 && nearest_zone_distance > 0);
   bool zone_very_near = (nearest_zone_distance < 50 && nearest_zone_distance > 0);

   if(has_nearby_zone)
     {
      // Не подтягиваем слишком близко к зоне ликвидности
      double zone_buffer = point * 20;  // 20 пунктов буфера
      if(nearest_zone_distance * point < trailing_distance + zone_buffer)
        {
         // Уменьшаем дистанцию, чтобы не дойти до зоны
         trailing_distance = MathMax(point * 30, nearest_zone_distance * point * 0.6);
         Log("[Adapt] Коррекция дистанции из-за зоны " + zone_type +
             " (" + DoubleToString(nearest_zone_distance, 0) + " пп)");
        }
     }

// 3.4. Минимальная и максимальная дистанция
   double min_distance = MathMax(point * 20, spread * 1.5);
   double max_distance = atr * 2.5;
   if(trailing_distance < min_distance)
      trailing_distance = min_distance;
   if(trailing_distance > max_distance)
      trailing_distance = max_distance;

// === 4. Расчёт шага подтяжки (адаптивный) ===
   double step_distance = atr * 0.3;
   if(step_distance < point * 15)
      step_distance = point * 15;

// === 5. Логика активации и подтяжки ===
   double new_sl = group.stop_loss;
   bool need_update = false;

// 5.1. Перенос в безубыток (после прохода 50% до ближайшей зоны)
   if(BreakevenAfterTP1 && !need_update)
     {
      bool should_breakeven = false;

      if(has_nearby_zone)
        {
         // Если прошли больше половины пути до зоны
         double progress = (nearest_zone_distance > 0) ?
                           (profit_pips / nearest_zone_distance) : 0;
         if(progress > 0.5 && profit_pips > spread_pips * 2)
            should_breakeven = true;
        }
      else
        {
         // Или если прибыль > 1.5 ATR
         if(profit_atr > 1.5)
            should_breakeven = true;
        }

      if(should_breakeven && current_rr > 0.8)
        {
         if(group.is_buy && group.stop_loss < entry_price)
           {
            new_sl = entry_price + spread;
            if(new_sl > group.stop_loss)
              {
               need_update = true;
               Log("[BE] " + group.group_name + " - перенос в БУ");
              }
           }
         if(!group.is_buy && group.stop_loss > entry_price)
           {
            new_sl = entry_price - spread;
            if(new_sl < group.stop_loss)
              {
               need_update = true;
               Log("[BE] " + group.group_name + " - перенос в БУ");
              }
           }
        }
     }

// 5.2. Активация трейлинга
   if(!need_update && profit_atr >= activation_profit / atr)
     {
      double candidate_sl = 0;

      if(group.is_buy)
        {
         candidate_sl = price_now - trailing_distance;
         // Не подтягиваем ближе чем к entry_price + буфер
         double min_sl = entry_price + point * 10;
         if(candidate_sl < min_sl)
            candidate_sl = min_sl;

         if(candidate_sl > group.stop_loss + step_distance)
           {
            new_sl = candidate_sl;
            need_update = true;
           }
        }
      else
        {
         candidate_sl = price_now + trailing_distance;
         double max_sl = entry_price - point * 10;
         if(candidate_sl > max_sl)
            candidate_sl = max_sl;

         if(candidate_sl < group.stop_loss - step_distance)
           {
            new_sl = candidate_sl;
            need_update = true;
           }
        }

      if(need_update)
        {
         Log("[Trail] " + group.group_name +
             " | Дист=" + DoubleToString(trailing_distance / point, 0) + " пп" +
             " | Прибыль=" + DoubleToString(profit_pips, 0) + " пп" +
             (has_nearby_zone ? " | Зона=" + DoubleToString(nearest_zone_distance, 0) + " пп" : ""));
        }
     }

// 5.3. Дополнительная защита: если подошли к зоне слишком близко
   if(!need_update && zone_very_near && profit_pips > 0)
     {
      // Забираем прибыль, не ждём разворота
      double tight_sl = 0;
      if(group.is_buy)
         tight_sl = nearest_zone_price - point * 15;
      else
         tight_sl = nearest_zone_price + point * 15;

      if((group.is_buy && tight_sl > group.stop_loss) ||
         (!group.is_buy && tight_sl < group.stop_loss))
        {
         new_sl = tight_sl;
         need_update = true;
         Log("[Zone] " + group.group_name + " - подтяжка к зоне " + zone_type);
        }
     }

// === 6. Применяем изменения ===
   if(need_update)
     {
      new_sl = NormalizeDouble(new_sl, GetDigits());

      for(int i = 0; i < group.order_count; i++)
        {
         if(OrderSelect(group.tickets[i], SELECT_BY_TICKET))
           {
            if(OrderCloseTime() == 0 && MathAbs(OrderStopLoss() - new_sl) > point)
              {
               if(OrderModify(OrderTicket(), OrderOpenPrice(), new_sl, OrderTakeProfit(), 0, clrNONE))
                 {
                  Log("  → Ордер " + IntegerToString(OrderTicket()) + ": SL " +
                      DoubleToString(OrderStopLoss(), GetDigits()) + " → " +
                      DoubleToString(new_sl, GetDigits()));
                 }
              }
           }
        }
      group.stop_loss = new_sl;
     }
  }

//+------------------------------------------------------------------+
//| Получение всех тикетов советника                                  |
//|                                                                   |
//| Заполняет массив tickets тикетами всех ордеров советника.         |
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   tickets[] — [OUT] массив для заполнения                        |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   Количество найденных тикетов                                    |
//+------------------------------------------------------------------+
int GetAllTicketByMagic(int &tickets[])
  {
   ArrayResize(tickets, 0);

   int our_magic = GetMagicNumber();
   int total = OrdersTotal();
   int count = 0;

   for(int i = total - 1; i >= 0; i--)
     {
      if(!OrderSelect(i, SELECT_BY_POS, MODE_TRADES))
         continue;

      if(OrderSymbol() == g_symbol && OrderMagicNumber() == our_magic)
        {
         ArrayResize(tickets, count + 1);
         tickets[count] = OrderTicket();
         count++;
        }
     }

   return count;
  }

//+------------------------------------------------------------------+
//| Проверка достаточности маржи для открытия позиции                |
//+------------------------------------------------------------------+
bool HasEnoughMargin(double lot_size)
  {
   double margin_required = MarketInfo(g_symbol, MODE_MARGINREQUIRED) * lot_size;
   return AccountFreeMargin() > margin_required * 1.3;
  }

//+------------------------------------------------------------------+
//| Проверка допустимости спреда для входа                           |
//+------------------------------------------------------------------+
bool IsSpreadAcceptable()
  {
   return (GetCurrentSpread() / GetPoint()) <= MaxSpreadPoints;
  }

//+------------------------------------------------------------------+
//| Проверка кулдауна после закрытия сделки                          |
//+------------------------------------------------------------------+
bool IsInCooldown()
  {
   if(CooldownMinutes <= 0 || g_last_close_time == 0)
      return false;
   return TimeCurrent() - g_last_close_time < CooldownMinutes * 60;
  }

//+------------------------------------------------------------------+
//| Проверка Mitigation (возврат цены к снятой зоне) - версия 2.0    |
//| Учитывает:                                                       |
//| 1. Подтверждение свечой (закрытие ВНУТРИ или ЗА зоной)           |
//| 2. Фильтр по времени (mitigation в течение N свечей после свипа) |
//| 3. ATR-фильтр (импульсность свечи retest)                        |
//| 4. Возврат к конкретной границе зоны                             |
//+------------------------------------------------------------------+
bool IsMitigationInProgress(LiquidityZone &zone, ENUM_TIMEFRAMES tf, int lookback = 15)
  {
// === 1. Базовая проверка: зона должна быть снята ===
   if(zone.is_active)
      return false;

   double point = GetPoint();
   double atr = GetATR(tf, 14, 0);
   if(atr <= 0)
      atr = point * 100;

   double local_price = GetCurrentBid();
   double zone_high = zone.price_top;
   double zone_low = zone.price_bottom;

// === 2. Определяем, какая граница нас интересует ===
// Для BUY (зона SSL): возврат к ВЕРХНЕЙ границе
// Для SELL (зона BSL): возврат к НИЖНЕЙ границе
   double target_boundary;
   bool is_bullish_zone = zone.is_bullish;  // true = BSL, false = SSL

   if(is_bullish_zone)
      target_boundary = zone_low;    // BSL: возврат к нижней границе (поддержка)
   else
      target_boundary = zone_high;   // SSL: возврат к верхней границе (сопротивление)

// === 3. Поиск свечи, на которой произошёл свип ===
   int sweep_bar = -1;
   datetime sweep_time = 0;

   for(int i = 1; i <= lookback; i++)
     {
      double high = GetHigh(tf, i);
      double low = GetLow(tf, i);
      double close = GetClose(tf, i);

      if(is_bullish_zone)  // BSL: свип вниз (пробили нижнюю границу)
        {
         if(low <= zone_low && close > zone_low)
           {
            sweep_bar = i;
            sweep_time = GetTime(tf, i);
            break;
           }
        }
      else  // SSL: свип вверх (пробили верхнюю границу)
        {
         if(high >= zone_high && close < zone_high)
           {
            sweep_bar = i;
            sweep_time = GetTime(tf, i);
            break;
           }
        }
     }

   if(sweep_bar < 0)
      return false;

// === 4. Проверка: цена вернулась к целевой границе ===
   bool price_at_boundary = false;

   if(is_bullish_zone)  // BSL: цена вернулась к нижней границе (или выше)
     {
      price_at_boundary = (local_price >= zone_low - point * 5 &&
                           local_price <= zone_low + atr * 0.3);
     }
   else  // SSL: цена вернулась к верхней границе (или ниже)
     {
      price_at_boundary = (local_price <= zone_high + point * 5 &&
                           local_price >= zone_high - atr * 0.3);
     }

   if(!price_at_boundary)
      return false;

// === 5. Проверка времени: mitigation должен быть СВЕЖИМ ===
// (не старше lookback свечей после свипа)
   datetime current_time = GetTime(tf, 0);
   if(current_time - sweep_time > lookback * GetPeriodSeconds(tf))
      return false;

// === 6. Поиск свечи mitigation (свеча, которая вернулась к зоне) ===
   int mitigation_bar = -1;
   double mitigation_body = 0;
   double mitigation_close = 0;

   for(int i = sweep_bar - 1; i >= 0; i--)
     {
      double high = GetHigh(tf, i);
      double low = GetLow(tf, i);
      double close = GetClose(tf, i);
      double open = GetOpen(tf, i);

      bool hit_boundary = false;

      if(is_bullish_zone)  // BSL: свеча коснулась нижней границы
        {
         hit_boundary = (low <= zone_low + point * 5 &&
                         high >= zone_low - point * 5);
        }
      else  // SSL: свеча коснулась верхней границы
        {
         hit_boundary = (high >= zone_high - point * 5 &&
                         low <= zone_high + point * 5);
        }

      if(hit_boundary)
        {
         mitigation_bar = i;
         mitigation_body = MathAbs(close - open);
         mitigation_close = close;
         break;
        }
     }

   if(mitigation_bar < 0)
      return false;

// === 7. ATR-фильтр: свеча mitigation должна быть импульсной ===
// (тело свечи > 30% ATR или > 50% от полного диапазона свечи)
   double candle_range = GetHigh(tf, mitigation_bar) - GetLow(tf, mitigation_bar);
   bool is_impulsive = false;

   if(candle_range > 0)
     {
      double body_ratio = mitigation_body / candle_range;
      if(body_ratio > 0.5 || mitigation_body > atr * 0.3)
         is_impulsive = true;
     }

   if(!is_impulsive)
      return false;

// === 8. Подтверждение закрытием свечи ===
// Цель: свеча должна закрыться ВНУТРИ зоны или за её пределами в правильном направлении
   bool close_confirmation = false;

   if(is_bullish_zone)  // BSL: ожидаем закрытие ВЫШЕ зоны
     {
      close_confirmation = (mitigation_close >= zone_low - point * 5);
     }
   else  // SSL: ожидаем закрытие НИЖЕ зоны
     {
      close_confirmation = (mitigation_close <= zone_high + point * 5);
     }

   if(!close_confirmation)
      return false;

// === 9. ВСЕ ПРОВЕРКИ ПРОЙДЕНЫ ===
   Log("[Mitigation] Обнаружен retest зоны " + (is_bullish_zone ? "BSL" : "SSL") +
       " | Свип на баре " + IntegerToString(sweep_bar) +
       " | Mitigation на баре " + IntegerToString(mitigation_bar) +
       " | Тело свечи: " + DoubleToString(mitigation_body / point, 0) + " пп");

   return true;
  }

//+------------------------------------------------------------------+
//| ПРАВИЛЬНЫЙ РАСЧЁТ УРОВНЕЙ PREMIUM/DISCOUNT (полностью новый)     |
//+------------------------------------------------------------------+
void CalculatePremiumDiscountLevels(ENUM_TIMEFRAMES tf, double &discount, double &premium, double &equilibrium)
  {
   discount = 0;
   premium = 0;
   equilibrium = 0;

// Находим глобальные свинги
   double highs[], lows[];
   int high_bars[], low_bars[];
   datetime high_times[], low_times[];

   FindAllSwings(tf, SwingStrength, true, highs, high_bars, high_times, 0, 200);
   FindAllSwings(tf, SwingStrength, false, lows, low_bars, low_times, 0, 200);

   if(ArraySize(highs) < 2 || ArraySize(lows) < 2)
     {
      // Если не нашли свингов, используем последние 100 баров
      double highest = -999999;
      double lowest = 999999;
      for(int i = 0; i < 100; i++)
        {
         double h = GetHigh(tf, i);
         double l = GetLow(tf, i);
         if(h > highest)
            highest = h;
         if(l < lowest)
            lowest = l;
        }
      if(highest > lowest)
        {
         double range = highest - lowest;
         discount = highest - range * (DiscountLevel / 100.0);
         equilibrium = highest - range * (EquilibriumLevel / 100.0);
         premium = highest - range * (PremiumLevel / 100.0);
        }
      return;
     }

// Берём последние значимые свинги
   double swing_high = highs[ArraySize(highs) - 1];
   double swing_low = lows[ArraySize(lows) - 1];

// Защита от некорректных значений
   if(swing_high <= swing_low)
     {
      discount = 0;
      premium = 0;
      equilibrium = 0;
      return;
     }

   double range = swing_high - swing_low;

   discount = swing_high - range * (DiscountLevel / 100.0);
   equilibrium = swing_high - range * (EquilibriumLevel / 100.0);
   premium = swing_high - range * (PremiumLevel / 100.0);
  }

//+------------------------------------------------------------------+
//| ОПРЕДЕЛЕНИЕ ЦЕНОВОЙ ЗОНЫ (без использования старых функций)      |
//+------------------------------------------------------------------+
ENUM_PRICE_ZONE DeterminePriceZone(ENUM_TIMEFRAMES tf)
  {
   if(!UsePremiumDiscountFilter)
      return ZONE_UNKNOWN;

   double discount = 0, premium = 0, equilibrium = 0;
   CalculatePremiumDiscountLevels(tf, discount, premium, equilibrium);

   if(discount <= 0 || premium <= 0)
      return ZONE_UNKNOWN;

   double local_price = GetCurrentBid();

   if(local_price <= discount)
      return ZONE_DISCOUNT;
   else
      if(local_price >= premium)
         return ZONE_PREMIUM;
      else
         if(local_price > discount && local_price < premium)
            return ZONE_EQUILIBRIUM;

   return ZONE_UNKNOWN;
  }

//+------------------------------------------------------------------+
//| ПРОВЕРКА НАПРАВЛЕНИЯ ДЛЯ PREMIUM/DISCOUNT (исправленная)         |
//+------------------------------------------------------------------+
bool IsValidZoneForDirection(bool is_buy, ENUM_TIMEFRAMES tf)
  {
   if(!UsePremiumDiscountFilter)
      return true;

// Получаем тренд из структуры
   ENUM_MARKET_STRUCTURE trend = GetCurrentTrend(g_htf_structure);

// Определяем зону
   ENUM_PRICE_ZONE zone = DeterminePriceZone(tf);

   if(zone == ZONE_UNKNOWN)
      return true; // Неопределённая зона - пропускаем

// ----- КЛЮЧЕВАЯ ЛОГИКА SMART MONEY CONCEPTS -----

   if(is_buy)
     {
      // По классическому SMC: покупки только в DISCOUNT или EQUILIBRIUM
      // Никогда не покупаем в PREMIUM (там продают крупные игроки)
      if(zone == ZONE_PREMIUM)
        {
         if(EnableLogging)
            Log("[ZoneFilter] BUY отклонён: цена в Premium зоне");
         return false;
        }
      return true;
     }
   else // is_sell
     {
      // По классическому SMC: продажи только в PREMIUM или EQUILIBRIUM
      // Никогда не продаём в DISCOUNT (там покупают крупные игроки)
      if(zone == ZONE_DISCOUNT)
        {
         if(EnableLogging)
            Log("[ZoneFilter] SELL отклонён: цена в Discount зоне");
         return false;
        }
      return true;
     }
  }

//+------------------------------------------------------------------+
//| БЕЗОПАСНАЯ РАБОТА С ОРДЕРАМИ (замена всех прямых OrderSelect)    |
//+------------------------------------------------------------------+
bool SelectOrderByTicket(int ticket, int select_mode = SELECT_BY_TICKET, int pool = MODE_TRADES)
  {
   if(ticket <= 0)
      return false;
   return OrderSelect(ticket, select_mode, pool);
  }

//+------------------------------------------------------------------+
//|                                                                  |
//+------------------------------------------------------------------+
bool SelectOrderByIndex(int index, int pool = MODE_TRADES)
  {
   if(index < 0)
      return false;
   return OrderSelect(index, SELECT_BY_POS, pool);
  }

//+------------------------------------------------------------------+
//| ПЕРЕРАБОТАННЫЙ ПОДСЧЁТ ОРДЕРОВ (было CountOurOrders)             |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//| Подсчёт всех ордеров советника (безопасный перебор)               |
//|                                                                   |
//| Ищет все ордера с нашим Magic Number на текущем символе.          |
//| Работает в цикле с защитой от бесконечного зависания.            |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   Количество ордеров, принадлежащих советнику                     |
//+------------------------------------------------------------------+
int CountOurOrdersSafe()
  {
   int count = 0;
   int our_magic = GetMagicNumber();
   int total = OrdersTotal();

// Защита от бесконечного цикла
   int max_iterations = total + 10;
   int iterations = 0;

   for(int i = total - 1; i >= 0; i--)
     {
      iterations++;
      if(iterations > max_iterations)
        {
         Log("✗ CountOurOrdersSafe: превышен лимит итераций");
         break;
        }

      if(!OrderSelect(i, SELECT_BY_POS, MODE_TRADES))
         continue;

      if(OrderSymbol() == g_symbol && OrderMagicNumber() == our_magic)
        {
         count++;
        }
     }

   return count;
  }

//+------------------------------------------------------------------+
//| Подсчёт ордеров советника по типу                                 |
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   cmd — OP_BUY или OP_SELL (-1 = все типы)                       |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   Количество ордеров заданного типа                               |
//+------------------------------------------------------------------+
int CountOurOrdersByType(int cmd = -1)
  {
   int count = 0;
   int our_magic = GetMagicNumber();
   int total = OrdersTotal();

   for(int i = total - 1; i >= 0; i--)
     {
      if(!OrderSelect(i, SELECT_BY_POS, MODE_TRADES))
         continue;

      if(OrderSymbol() != g_symbol)
         continue;

      if(OrderMagicNumber() != our_magic)
         continue;

      if(cmd == -1 || OrderType() == cmd)
        {
         count++;
        }
     }

   return count;
  }

//+------------------------------------------------------------------+
//| ПЕРЕРАБОТАННАЯ ПРОВЕРКА АКТИВНЫХ ОРДЕРОВ                         |
//+------------------------------------------------------------------+
bool HasActiveOrdersSafe()
  {
   for(int i = 0; i < OrdersTotal(); i++)
     {
      if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES))
        {
         if(OrderSymbol() == g_symbol && IsOurOrder(OrderTicket()))
            return true;
        }
     }
   return false;
  }

//+------------------------------------------------------------------+
//| ПРОВЕРКА НАЛИЧИЯ СВЕЖЕГО MITIGATION ПОСЛЕ СВИПА                  |
//+------------------------------------------------------------------+
bool HasRecentMitigation(LiquidityZone &swept_zone, ENUM_TIMEFRAMES tf)
  {
// Проверяем, была ли зона снята
   if(swept_zone.is_active)
      return false;

// Проверяем митигейшн
   return IsMitigationInProgress(swept_zone, tf, 10);
  }

//+------------------------------------------------------------------+
//| ПОИСК ЛУЧШЕЙ СНЯТОЙ ЗОНЫ С MITIGATION                            |
//+------------------------------------------------------------------+
bool FindSweptZoneWithMitigation(LiquidityZone &zones[], bool is_bullish_zone, ENUM_TIMEFRAMES tf, LiquidityZone &result)
  {
   datetime current_time = GetGMTTime();

   for(int i = 0; i < ArraySize(zones); i++)
     {
      // Зона должна быть снята
      if(zones[i].is_active)
         continue;

      // Проверяем направление
      if(zones[i].is_bullish != is_bullish_zone)
         continue;

      // Проверяем свежесть свипа (не старше 24 часов)
      if(current_time - zones[i].last_time > 86400)
         continue;

      // Проверяем наличие митигейшн
      if(IsMitigationInProgress(zones[i], tf, 8))
        {
         result = zones[i];
         return true;
        }
     }

   return false;
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 5 🔹                              |
//|             ПОИСК СВИНГ-УРОВНЕЙ И ЗОН ЛИКВИДНОСТИ                |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Поиск всех свингов (локальных экстремумов) на указанном ТФ       |
//+------------------------------------------------------------------+
void FindAllSwings(ENUM_TIMEFRAMES tf, int strength, bool find_highs,
                   double &prices[], int &bar_indices[], datetime &times[],
                   int start_bar = 0, int max_depth = 1000)
  {
   ArrayResize(prices, 0);
   ArrayResize(bar_indices, 0);
   ArrayResize(times, 0);
   int total_bars = GetBars(tf);
   int search_start = start_bar + strength;
   int search_end = MathMin(search_start + max_depth, total_bars - strength - 1);
   if(search_end <= search_start)
      return;

   double atr = GetATR(tf, 14, 0);
   if(atr <= 0)
      atr = 0.0001;

   for(int i = search_start; i < search_end; i++)
     {
      double local_price = find_highs ? GetHigh(tf, i) : GetLow(tf, i);
      if(local_price <= 0)
         continue;

      bool is_swing = true;
      for(int j = 1; j <= strength; j++)
        {
         double left_price = find_highs ? GetHigh(tf, i - j) : GetLow(tf, i - j);
         double right_price = find_highs ? GetHigh(tf, i + j) : GetLow(tf, i + j);
         if(find_highs)
           {
            if(left_price >= local_price || right_price >= local_price)
              {
               is_swing = false;
               break;
              }
           }
         else
           {
            if(left_price <= local_price || right_price <= local_price)
              {
               is_swing = false;
               break;
              }
           }
        }
      if(!is_swing)
         continue;

      if(find_highs)
        {
         double min_close = GetClose(tf, i + 1);
         for(int k = 2; k <= strength; k++)
           {
            double c = GetClose(tf, i + k);
            if(c < min_close)
               min_close = c;
           }
         if((local_price - min_close) / atr < MinSwingPullbackATR)
            continue;
        }
      else
        {
         double max_close = GetClose(tf, i + 1);
         for(int k = 2; k <= strength; k++)
           {
            double c = GetClose(tf, i + k);
            if(c > max_close)
               max_close = c;
           }
         if((max_close - local_price) / atr < MinSwingPullbackATR)
            continue;
        }

      int size = ArraySize(prices);
      ArrayResize(prices, size + 1);
      ArrayResize(bar_indices, size + 1);
      ArrayResize(times, size + 1);
      prices[size] = local_price;
      bar_indices[size] = i;
      times[size] = GetTime(tf, i);
     }
  }

//+------------------------------------------------------------------+
//| Группировка близких свингов в зоны ликвидности                   |
//+------------------------------------------------------------------+
void GroupSwingsToZones(double &prices[], datetime &times[], bool is_bullish, LiquidityZone &zones[])
  {
   ArrayResize(zones, 0);
   int count = ArraySize(prices);
   if(count == 0)
      return;

   for(int i = 0; i < count - 1; i++)
     {
      for(int j = i + 1; j < count; j++)
        {
         if(times[i] > times[j])
           {
            double tmp_p = prices[i];
            prices[i] = prices[j];
            prices[j] = tmp_p;
            datetime tmp_t = times[i];
            times[i] = times[j];
            times[j] = tmp_t;
           }
        }
     }

   double tolerance = EqualTolerancePoints * GetPoint();
   bool grouped[];
   ArrayResize(grouped, count);
   ArrayInitialize(grouped, false);

   for(int i = 0; i < count; i++)
     {
      if(grouped[i])
         continue;
      LiquidityZone zone;
      zone.price_top = prices[i] + tolerance;
      zone.price_bottom = prices[i] - tolerance;
      zone.price_center = prices[i];
      zone.swing_count = 1;
      zone.is_active = true;
      zone.is_bullish = is_bullish;
      zone.first_time = times[i];
      zone.last_time = times[i];

      for(int j = i + 1; j < count; j++)
        {
         if(grouped[j])
            continue;
         if(MathAbs(prices[j] - zone.price_center) <= tolerance)
           {
            grouped[j] = true;
            zone.swing_count++;
            if(prices[j] > zone.price_center + tolerance)
               zone.price_top = prices[j] + tolerance;
            if(prices[j] < zone.price_center - tolerance)
               zone.price_bottom = prices[j] - tolerance;
            zone.price_center = (zone.price_top + zone.price_bottom) / 2.0;
            zone.last_time = times[j];
           }
        }
      grouped[i] = true;
      int size = ArraySize(zones);
      ArrayResize(zones, size + 1);
      zones[size] = zone;
     }

   for(int i = 0; i < ArraySize(zones) - 1; i++)
     {
      for(int j = i + 1; j < ArraySize(zones); j++)
        {
         if(is_bullish)
           {
            if(zones[i].price_center < zones[j].price_center)
              {
               LiquidityZone tmp = zones[i];
               zones[i] = zones[j];
               zones[j] = tmp;
              }
           }
         else
           {
            if(zones[i].price_center > zones[j].price_center)
              {
               LiquidityZone tmp = zones[i];
               zones[i] = zones[j];
               zones[j] = tmp;
              }
           }
        }
     }
  }

//+------------------------------------------------------------------+
//| Получение всех зон ликвидности BSL и SSL на указанном ТФ         |
//+------------------------------------------------------------------+
void GetAllLiquidityZones(ENUM_TIMEFRAMES tf, int strength, LiquidityZone &bsl_zones[], LiquidityZone &ssl_zones[],
                          int start_bar = 0, int max_depth = 1000)
  {
   double bsl_prices[], ssl_prices[];
   int bsl_bars[], ssl_bars[];
   datetime bsl_times[], ssl_times[];
   FindAllSwings(tf, strength, true, bsl_prices, bsl_bars, bsl_times, start_bar, max_depth);
   FindAllSwings(tf, strength, false, ssl_prices, ssl_bars, ssl_times, start_bar, max_depth);
   GroupSwingsToZones(bsl_prices, bsl_times, true, bsl_zones);
   GroupSwingsToZones(ssl_prices, ssl_times, false, ssl_zones);
  }

//+------------------------------------------------------------------+
//| Обновление статуса зон ликвидности (BSL/SSL Sweep Detection)      |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//| Обновление статуса зон ликвидности (BSL/SSL Sweep Detection)      |
//|                                                                   |
//| ЛОГИКА:                                                           |
//|   BSL (is_bullish = true)  — стоп-лоссы продавцов НАД ценой.     |
//|        Снимается когда цена пробивает ВЕРХНЮЮ границу зоны.      |
//|                                                                   |
//|   SSL (is_bullish = false) — стоп-лоссы покупателей ПОД ценой.   |
//|        Снимается когда цена пробивает НИЖНЮЮ границу зоны.       |
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   tf           — таймфрейм для анализа                            |
//|   zones[]      — массив зон ликвидности                           |
//|   lookback_bars — сколько баров назад проверять (по умолчанию 30)|
//+------------------------------------------------------------------+
void UpdateLiquidityZonesStatus(ENUM_TIMEFRAMES tf, LiquidityZone &zones[], int lookback_bars = 30)
  {
// Проверяем что массив не пуст
   if(ArraySize(zones) == 0)
      return;

// Получаем high и low предыдущей закрытой свечи
   double current_high = GetHigh(tf, 1);
   double current_low  = GetLow(tf, 1);
   datetime current_time = GetTime(tf, 1);

// Проверяем корректность полученных данных
   if(current_high <= 0 || current_low <= 0)
     {
      Log("⚠ UpdateLiquidityZonesStatus: некорректные данные High/Low");
      return;
     }

// Проходим по всем зонам
   for(int i = 0; i < ArraySize(zones); i++)
     {
      // Пропускаем уже снятые зоны
      if(zones[i].is_swept)
         continue;

      // Пропускаем неактивные зоны
      if(!zones[i].is_active)
         continue;

      bool swept = false;
      double sweep_price = 0;

      // ================================================================
      // BSL (Бычья зона ликвидности = стопы продавцов наверху)
      // Зона is_bullish = true
      // Снятие: цена пробила ВЕРХНЮЮ границу зоны
      // ================================================================
      if(zones[i].is_bullish)
        {
         // Основное условие: high свечи >= верхняя граница зоны
         // Дополнительно: low свечи должен быть <= верхняя граница (тело свечи касается границы)
         if(current_high >= zones[i].price_top && current_low <= zones[i].price_top)
           {
            swept = true;
            sweep_price = zones[i].price_top;
           }
        }
      // ================================================================
      // SSL (Медвежья зона ликвидности = стопы покупателей внизу)
      // Зона is_bullish = false
      // Снятие: цена пробила НИЖНЮЮ границу зоны
      // ================================================================
      else
        {
         // Основное условие: low свечи <= нижняя граница зоны
         // Дополнительно: high свечи должен быть >= нижняя граница (тело свечи касается границы)
         if(current_low <= zones[i].price_bottom && current_high >= zones[i].price_bottom)
           {
            swept = true;
            sweep_price = zones[i].price_bottom;
           }
        }

      // Если зона снята — обновляем её статус
      if(swept)
        {
         zones[i].is_swept = true;
         zones[i].sweep_time = current_time;
         zones[i].sweep_price = sweep_price;
         zones[i].is_active = false;  // Деактивируем зону после снятия

         // Логируем событие
         string zone_type = zones[i].is_bullish ? "BSL" : "SSL";
         Log("✓ Зона " + zone_type + " #" + IntegerToString(i) +
             " снята на цене " + DoubleToString(sweep_price, GetDigits()) +
             " | Время: " + TimeToString(current_time) +
             " | Top: " + DoubleToString(zones[i].price_top, GetDigits()) +
             " | Bottom: " + DoubleToString(zones[i].price_bottom, GetDigits()));
        }
     }
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 6 🔹                              |
//|                СТРУКТУРА РЫНКА (BOS + CHoCH)                     |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Анализ рыночной структуры: поиск BOS и CHoCH                     |
//+------------------------------------------------------------------+
void AnalyzeMarketStructure(ENUM_TIMEFRAMES tf, int strength, StructurePoint &structure[],
                            int start_bar = 0, int max_depth = 1000)
  {
   double highs[], lows[];
   int high_bars[], low_bars[];
   datetime high_times[], low_times[];
   FindAllSwings(tf, strength, true, highs, high_bars, high_times, start_bar, max_depth);
   FindAllSwings(tf, strength, false, lows, low_bars, low_times, start_bar, max_depth);

   int total_swings = ArraySize(highs) + ArraySize(lows);
   ArrayResize(structure, total_swings);

   int idx = 0;
   for(int i = 0; i < ArraySize(highs); i++)
     {
      structure[idx].price = highs[i];
      structure[idx].bar_index = high_bars[i];
      structure[idx].time = high_times[i];
      structure[idx].is_high = true;
      structure[idx].is_bos = false;
      structure[idx].is_choch = false;
      idx++;
     }
   for(int i = 0; i < ArraySize(lows); i++)
     {
      structure[idx].price = lows[i];
      structure[idx].bar_index = low_bars[i];
      structure[idx].time = low_times[i];
      structure[idx].is_high = false;
      structure[idx].is_bos = false;
      structure[idx].is_choch = false;
      idx++;
     }

   for(int i = 0; i < total_swings - 1; i++)
      for(int j = i + 1; j < total_swings; j++)
         if(structure[i].time > structure[j].time)
           {
            StructurePoint tmp = structure[i];
            structure[i] = structure[j];
            structure[j] = tmp;
           }

   double last_swing_high = 0, last_swing_low = 0;
   ENUM_MARKET_STRUCTURE current_structure = STRUCTURE_UNDEFINED;

   for(int i = 0; i < total_swings; i++)
     {
      if(structure[i].is_high)
        {
         if(current_structure == STRUCTURE_BULLISH && structure[i].price > last_swing_high)
           {
            structure[i].is_bos = true;
            last_swing_high = structure[i].price;
           }
         else
            if(current_structure == STRUCTURE_BEARISH && structure[i].price > last_swing_high)
              {
               structure[i].is_choch = true;
               current_structure = STRUCTURE_BULLISH;
               last_swing_high = structure[i].price;
              }
            else
               if(current_structure == STRUCTURE_UNDEFINED)
                  last_swing_high = structure[i].price;
               else
                  if(structure[i].price > last_swing_high)
                     last_swing_high = structure[i].price;
        }
      else
        {
         if(current_structure == STRUCTURE_BEARISH && structure[i].price < last_swing_low)
           {
            structure[i].is_bos = true;
            last_swing_low = structure[i].price;
           }
         else
            if(current_structure == STRUCTURE_BULLISH && structure[i].price < last_swing_low)
              {
               structure[i].is_choch = true;
               current_structure = STRUCTURE_BEARISH;
               last_swing_low = structure[i].price;
              }
            else
               if(current_structure == STRUCTURE_UNDEFINED)
                  last_swing_low = structure[i].price;
               else
                  if(structure[i].price < last_swing_low)
                     last_swing_low = structure[i].price;
        }
      if(current_structure == STRUCTURE_UNDEFINED && last_swing_high > 0 && last_swing_low > 0)
         current_structure = (last_swing_high > last_swing_low) ? STRUCTURE_BULLISH : STRUCTURE_BEARISH;
     }
  }

//+------------------------------------------------------------------+
//| Определение текущего тренда на основе структуры                  |
//+------------------------------------------------------------------+
ENUM_MARKET_STRUCTURE GetCurrentTrend(StructurePoint &structure[])
  {
   int total = ArraySize(structure);
   if(total < 2)
      return STRUCTURE_UNDEFINED;

   int count = MathMin(5, total);
   int highs_above = 0, lows_above = 0;
   double prev_high = 0, prev_low = 0;

   for(int i = total - count; i < total; i++)
     {
      if(structure[i].is_high)
        {
         if(prev_high > 0 && structure[i].price > prev_high)
            highs_above++;
         prev_high = structure[i].price;
        }
      else
        {
         if(prev_low > 0 && structure[i].price > prev_low)
            lows_above++;
         prev_low = structure[i].price;
        }
     }
   if(highs_above >= 1 && lows_above >= 1)
      return STRUCTURE_BULLISH;

   int highs_below = 0, lows_below = 0;
   prev_high = 0;
   prev_low = 0;
   for(int i = total - count; i < total; i++)
     {
      if(structure[i].is_high)
        {
         if(prev_high > 0 && structure[i].price < prev_high)
            highs_below++;
         prev_high = structure[i].price;
        }
      else
        {
         if(prev_low > 0 && structure[i].price < prev_low)
            lows_below++;
         prev_low = structure[i].price;
        }
     }
   if(highs_below >= 1 && lows_below >= 1)
      return STRUCTURE_BEARISH;

   return STRUCTURE_UNDEFINED;
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 7 🔹                              |
//|                      FVG и ORDER BLOCKS                          |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Поиск всех Fair Value Gap (FVG) на указанном ТФ                  |
//+------------------------------------------------------------------+
void FindFVGs(ENUM_TIMEFRAMES tf, FVG &fvgs[], int start_bar = 1, int max_depth = 500)
  {
   ArrayResize(fvgs, 0);
   int total_bars = GetBars(tf);
   
   // Минимум start_bar = 1 (нельзя использовать незакрытую свечу 0)
   int actual_start = MathMax(start_bar, 1);
   int search_end = MathMin(actual_start + max_depth, total_bars - 3);

   for(int i = actual_start; i < search_end; i++)
     {
      // Свеча 1 (новая, i), Свеча 2 (средняя, i+1), Свеча 3 (старая, i+2)
      double high0 = GetHigh(tf, i),   low0 = GetLow(tf, i);       // свеча 1
      double high1 = GetHigh(tf, i+1), low1 = GetLow(tf, i+1);     // свеча 2
      double high2 = GetHigh(tf, i+2), low2 = GetLow(tf, i+2);     // свеча 3

      // Бычий FVG: low старой свечи (i+2) > high новой свечи (i)
      // Разрыв между тенями — средняя свеча НЕ перекрывает
      if(low2 > high0)
        {
         int size = ArraySize(fvgs);
         ArrayResize(fvgs, size + 1);
         fvgs[size].price_top    = low2;      // верх FVG = low свечи 3
         fvgs[size].price_bottom = high0;     // низ FVG = high свечи 1
         fvgs[size].bar_index    = i;
         fvgs[size].time         = GetTime(tf, i);
         fvgs[size].is_bullish   = true;
         fvgs[size].is_filled    = false;
        }

      // Медвежий FVG: low новой свечи (i) > high старой свечи (i+2)
      else if(low0 > high2)
        {
         int size = ArraySize(fvgs);
         ArrayResize(fvgs, size + 1);
         fvgs[size].price_top    = low0;      // верх FVG = low свечи 1
         fvgs[size].price_bottom = high2;     // низ FVG = high свечи 3
         fvgs[size].bar_index    = i;
         fvgs[size].time         = GetTime(tf, i);
         fvgs[size].is_bullish   = false;
         fvgs[size].is_filled    = false;
        }
     }
  }

//+------------------------------------------------------------------+
//| Обновление статуса FVG (проверка заполнения)                     |
//+------------------------------------------------------------------+
void UpdateFVGStatus(ENUM_TIMEFRAMES tf, FVG &fvgs[])
  {
   for(int f = 0; f < ArraySize(fvgs); f++)
     {
      if(fvgs[f].is_filled)
         continue;
      for(int i = fvgs[f].bar_index - 1; i >= 0; i--)
        {
         if(GetHigh(tf, i) >= fvgs[f].price_bottom && GetLow(tf, i) <= fvgs[f].price_top)
           {
            fvgs[f].is_filled = true;
            break;
           }
        }
     }
  }

//+------------------------------------------------------------------+
//| Поиск ближайшего незаполненного FVG (ИСПРАВЛЕНО)                  |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//| Поиск ближайшего незаполненного Fair Value Gap (FVG)              |
//|                                                                   |
//| ЛОГИКА:                                                           |
//|   Для BUY (is_buy = true):  ищем БЫЧИЙ FVG ПОД текущей ценой    |
//|   Для SELL (is_buy = false): ищем МЕДВЕЖИЙ FVG НАД текущей ценой|
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   fvgs[]        — массив всех FVG                                |
//|   current_price — текущая цена для сравнения                     |
//|   is_buy        — направление поиска (true = BUY, false = SELL)  |
//|   nearest_fvg   — [OUT] найденный ближайший FVG                  |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   true  — найден хотя бы один незаполненный FVG                  |
//|   false — подходящих FVG нет                                     |
//+------------------------------------------------------------------+
bool FindNearestUnfilledFVG(FVG &fvgs[], double local_price, bool is_buy, FVG &nearest_fvg)
  {
   double min_distance = DBL_MAX;
   bool found = false;

// Обнуляем результат
   ZeroMemory(nearest_fvg);

// Проходим по всем FVG
   for(int i = 0; i < ArraySize(fvgs); i++)
     {
      // Пропускаем уже заполненные FVG
      if(fvgs[i].is_filled)
         continue;

      // ================================================================
      // Для BUY: ищем БЫЧИЙ FVG ПОД текущей ценой
      // ================================================================
      if(is_buy)
        {
         // FVG должен быть бычьим и находиться ниже цены
         if(fvgs[i].is_bullish && fvgs[i].price_top < local_price)
           {
            double distance = local_price - fvgs[i].price_bottom;

            if(distance < min_distance)
              {
               min_distance = distance;
               nearest_fvg = fvgs[i];
               found = true;
              }
           }
        }
      // ================================================================
      // Для SELL: ищем МЕДВЕЖИЙ FVG НАД текущей ценой
      // ================================================================
      else
        {
         // FVG должен быть медвежьим и находиться выше цены
         if(!fvgs[i].is_bullish && fvgs[i].price_bottom > local_price)
           {
            double distance = fvgs[i].price_top - local_price;

            if(distance < min_distance)
              {
               min_distance = distance;
               nearest_fvg = fvgs[i];
               found = true;
              }
           }
        }
     }

// Если нашли — логируем
   if(found)
     {
      string fvg_type = nearest_fvg.is_bullish ? "Бычий" : "Медвежий";
      Log("✓ Найден ближайший " + fvg_type + " FVG | " +
          "Цена: " + DoubleToString(current_price, GetDigits()) +
          " | Расстояние: " + DoubleToString(min_distance / GetPoint(), 0) + " пунктов" +
          " | FVG Top: " + DoubleToString(nearest_fvg.price_top, GetDigits()) +
          " | FVG Bottom: " + DoubleToString(nearest_fvg.price_bottom, GetDigits()));
     }

   return found;
  }

//+------------------------------------------------------------------+
//| ПРАВИЛЬНЫЙ ПОИСК ORDER BLOCKS                                    |
//+------------------------------------------------------------------+
void FindOrderBlocksCorrect(ENUM_TIMEFRAMES tf, int strength, OrderBlock &obs[], int start_bar = 1, int max_depth = 500)
  {
   ArrayResize(obs, 0);
   int total_bars = GetBars(tf);
   
   int actual_start = MathMax(start_bar, 2);  // Минимум 2, т.к. смотрим бары после OB
   int search_end = MathMin(actual_start + max_depth, total_bars - 5);

   for(int i = actual_start; i < search_end; i++)
     {
      double close_i = GetClose(tf, i);
      double open_i  = GetOpen(tf, i);
      double high_i  = GetHigh(tf, i);
      double low_i   = GetLow(tf, i);

      // БЫЧИЙ Order Block = медвежья свеча (close < open)
      // Смарт-мани набирают длинную позицию на падении
      if(close_i < open_i)
        {
         // Ищем рост ПОСЛЕ этой свечи (бары i-1 ... i-5 — более новые)
         double max_close_after = close_i;
         int bars_after = 0;

         for(int j = i - 1; j >= MathMax(0, i - 5); j--)
           {
            double close_j = GetClose(tf, j);
            if(close_j > max_close_after)
               max_close_after = close_j;
            bars_after++;
           }

         double atr = GetATR(tf, 14, i);
         if(atr <= 0)
            atr = GetPoint() * 100;

         double rise = max_close_after - low_i;

         // Рост должен быть минимум 50% ATR за 2+ бара
         if(rise > atr * 0.5 && bars_after >= 2)
           {
            int size = ArraySize(obs);
            ArrayResize(obs, size + 1);
            obs[size].price_top  = high_i;
            obs[size].price_bottom = low_i;
            obs[size].open       = open_i;
            obs[size].close      = close_i;
            obs[size].bar_index  = i;
            obs[size].time       = GetTime(tf, i);
            obs[size].is_bullish = true;   // Бычий OB (ждём рост)
            obs[size].is_tested  = false;
            obs[size].is_broken  = false;
           }
        }

      // МЕДВЕЖИЙ Order Block = бычья свеча (close > open)
      // Смарт-мани раздают позицию на росте
      if(close_i > open_i)
        {
         double min_close_after = close_i;
         int bars_after = 0;

         for(int j = i - 1; j >= MathMax(0, i - 5); j--)
           {
            double close_j = GetClose(tf, j);
            if(close_j < min_close_after)
               min_close_after = close_j;
            bars_after++;
           }

         double atr = GetATR(tf, 14, i);
         if(atr <= 0)
            atr = GetPoint() * 100;

         double drop = high_i - min_close_after;

         if(drop > atr * 0.5 && bars_after >= 2)
           {
            int size = ArraySize(obs);
            ArrayResize(obs, size + 1);
            obs[size].price_top  = high_i;
            obs[size].price_bottom = low_i;
            obs[size].open       = open_i;
            obs[size].close      = close_i;
            obs[size].bar_index  = i;
            obs[size].time       = GetTime(tf, i);
            obs[size].is_bullish = false;  // Медвежий OB (ждём падение)
            obs[size].is_tested  = false;
            obs[size].is_broken  = false;
           }
        }
     }
  }

//+------------------------------------------------------------------+
//| Обновление статуса Order Block (тестирование/пробой)             |
//+------------------------------------------------------------------+
void UpdateOrderBlockStatus(ENUM_TIMEFRAMES tf, OrderBlock &obs[])
  {
   int total_bars = GetBars(tf);
   
   for(int o = 0; o < ArraySize(obs); o++)
     {
      // Пропускаем уже сломанные
      if(obs[o].is_broken)
         continue;
      
      // Проверяем максимум 500 баров от OB (оптимизация)
      int start_check = obs[o].bar_index - 1;
      int end_check   = MathMax(0, obs[o].bar_index - 500);
      
      for(int i = start_check; i >= end_check; i--)
        {
         double high  = GetHigh(tf, i);
         double low   = GetLow(tf, i);
         double close = GetClose(tf, i);
         
         // Тестирование: цена коснулась зоны OB (хотя бы частью бара)
         if(!obs[o].is_tested && low <= obs[o].price_top && high >= obs[o].price_bottom)
            obs[o].is_tested = true;
         
         // Проверка пробоя с подтверждением
         if(obs[o].is_bullish)
           {
            // Бычий OB сломан: закрытие ниже price_bottom + подтверждение
            if(close < obs[o].price_bottom && i > 0)
              {
               double close_next = GetClose(tf, i - 1);
               if(close_next < obs[o].price_bottom)
                 {
                  obs[o].is_broken = true;
                  break;
                 }
              }
           }
         else
           {
            // Медвежий OB сломан: закрытие выше price_top + подтверждение
            if(close > obs[o].price_top && i > 0)
              {
               double close_next = GetClose(tf, i - 1);
               if(close_next > obs[o].price_top)
                 {
                  obs[o].is_broken = true;
                  break;
                 }
              }
           }
        }
     }
  }

//+------------------------------------------------------------------+
//| Поиск ближайшего активного Order Block в направлении сделки      |
//+------------------------------------------------------------------+
bool FindNearestActiveOB(OrderBlock &obs[], double local_price, bool want_bullish, OrderBlock &nearest_ob)
  {
   // Инициализируем как "не найден"
   nearest_ob.is_broken = true;
   double min_distance = DBL_MAX;
   
   for(int i = 0; i < ArraySize(obs); i++)
     {
      // Пропускаем сломанные и несовпадающие по направлению
      if(obs[i].is_broken || obs[i].is_bullish != want_bullish)
         continue;
      
      // Расстояние до ближайшей границы диапазона OB
      double distance;
      if(local_price > obs[i].price_top)
         distance = local_price - obs[i].price_top;          // цена выше OB
      else if(local_price < obs[i].price_bottom)
         distance = obs[i].price_bottom - local_price;       // цена ниже OB
      else
         distance = 0.0;                                      // цена внутри OB
      
      if(distance < min_distance)
        {
         min_distance = distance;
         nearest_ob = obs[i];
        }
     }
   
   // Если nearest_ob.is_broken == true — ничего не нашли
   return !nearest_ob.is_broken;
  }

//+------------------------------------------------------------------+
//| Поиск Liquidity Voids (тела импульсных свечей)                   |
//+------------------------------------------------------------------+
void FindLiquidityVoids(ENUM_TIMEFRAMES tf, FVG &voids[], int start_bar = 1, int max_depth = 500)
  {
   ArrayResize(voids, 0);
   int total_bars = GetBars(tf);
   
   int actual_start = MathMax(start_bar, 1);
   int search_end = MathMin(actual_start + max_depth, total_bars - 1);

   for(int i = actual_start; i < search_end; i++)
     {
      double open  = GetOpen(tf, i);
      double close = GetClose(tf, i);
      double body  = MathAbs(close - open);
      double atr   = GetATR(tf, 14, i);

      // LV — только импульсные свечи (тело > 70% ATR)
      if(atr <= 0 || body < atr * 0.7)
         continue;

      bool is_mitigated = false;

      // Проверяем 5 баров ПОСЛЕ LV (i-1 ... i-5 — более новые)
      for(int j = i - 1; j >= MathMax(0, i - 5); j--)
        {
         double close_j = GetClose(tf, j);

         if(close > open)  // Бычий LV
           {
            // Цена закрылась внутри тела = mitigation
            if(close_j < close && close_j > open)
              {
               is_mitigated = true;
               break;
              }
           }
         else  // Медвежий LV
           {
            if(close_j > close && close_j < open)
              {
               is_mitigated = true;
               break;
              }
           }
        }

      if(!is_mitigated)
        {
         int size = ArraySize(voids);
         ArrayResize(voids, size + 1);
         voids[size].price_top    = MathMax(open, close);   // верх тела
         voids[size].price_bottom = MathMin(open, close);   // низ тела
         voids[size].bar_index    = i;
         voids[size].time         = GetTime(tf, i);
         voids[size].is_bullish   = (close > open);
         voids[size].is_filled    = false;
        }
     }
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 8 🔹                              |
//|                   ОЦЕНКА КАЧЕСТВА СИГНАЛА                        |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Оценка волатильности для уровня ТФ (соотношение HTF/LTF ATR)     |
//+------------------------------------------------------------------+
double CalculateVolatilityFitness(TFLevel &level)
  {
   double atr_htf = GetATR(level.htf, 14, 0), atr_ltf = GetATR(level.ltf, 14, 0);
   if(atr_htf <= 0 || atr_ltf <= 0)
      return 0;
   double ratio = atr_htf / atr_ltf;
   if(ratio >= 3 && ratio <= 5)
      return 25;
   if(ratio >= 2 && ratio <= 7)
      return 15;
   if(ratio >= 1.5 && ratio <= 10)
      return 8;
   return 3;
  }

//+------------------------------------------------------------------+
//| Оценка качества рыночной структуры (BOS/CHoCH)                   |
//+------------------------------------------------------------------+
double EvaluateStructureQuality(ENUM_TIMEFRAMES htf, int strength)
  {
   StructurePoint structure[];
   AnalyzeMarketStructure(htf, strength, structure);
   if(ArraySize(structure) < 3)
      return 5;
   int bos_count = 0, choch_count = 0;
   for(int i = MathMax(0, ArraySize(structure) - 20); i < ArraySize(structure); i++)
     {
      if(structure[i].is_bos)
         bos_count++;
      if(structure[i].is_choch)
         choch_count++;
     }
   double score = 0;
   if(bos_count >= 3)
      score += 10;
   else
      if(bos_count >= 1)
         score += 6;
      else
         score += 2;
   if(choch_count >= 2)
      score += 8;
   else
      if(choch_count >= 1)
         score += 5;
   if(GetCurrentTrend(structure) != STRUCTURE_UNDEFINED)
      score += 7;
   return MathMin(25, score);
  }

//+------------------------------------------------------------------+
//| Оценка качества ликвидности (наличие снятых зон)                 |
//+------------------------------------------------------------------+
double EvaluateLiquidityQuality(ENUM_TIMEFRAMES htf, ENUM_TIMEFRAMES ltf, int strength, double price)
  {
   LiquidityZone bsl_zones[], ssl_zones[];
   GetAllLiquidityZones(htf, strength, bsl_zones, ssl_zones);
   UpdateLiquidityZonesStatus(ltf, bsl_zones, 30);
   UpdateLiquidityZonesStatus(ltf, ssl_zones, 30);

   int swept_bsl = 0, swept_ssl = 0, active_bsl = 0, active_ssl = 0;
   for(int i = 0; i < ArraySize(bsl_zones); i++)
     {
      if(!bsl_zones[i].is_active)
         swept_bsl++;
      else
         active_bsl++;
     }
   for(int i = 0; i < ArraySize(ssl_zones); i++)
     {
      if(!ssl_zones[i].is_active)
         swept_ssl++;
      else
         active_ssl++;
     }

   double score = 0;
   if(swept_bsl > 0 && active_ssl > 0)
      score += 15;
   else
      if(swept_ssl > 0 && active_bsl > 0)
         score += 15;
      else
         if(swept_bsl > 0 || swept_ssl > 0)
            score += 8;
         else
            score += 2;
   if(active_bsl + active_ssl >= 3)
      score += 5;
   else
      if(active_bsl + active_ssl >= 1)
         score += 3;

   int max_swings = 0;
   for(int i = 0; i < ArraySize(bsl_zones); i++)
     {
      if(bsl_zones[i].swing_count > max_swings)
         max_swings = bsl_zones[i].swing_count;
     }
   for(int i = 0; i < ArraySize(ssl_zones); i++)
     {
      if(ssl_zones[i].swing_count > max_swings)
         max_swings = ssl_zones[i].swing_count;
     }
   if(max_swings >= 3)
      score += 5;
   else
      if(max_swings >= 2)
         score += 3;

   return MathMin(25, score);
  }

//+------------------------------------------------------------------+
//| Оценка качества паттернов (FVG и OB)                             |
//+------------------------------------------------------------------+
double EvaluatePatternQuality(ENUM_TIMEFRAMES ltf, double local_price)
  {
   double score = 0;
   FVG fvgs[];
   FindFVGs(ltf, fvgs, 0, 200);
   UpdateFVGStatus(ltf, fvgs);
   int unfilled = 0;
   for(int i = 0; i < ArraySize(fvgs); i++)
     {
      if(!fvgs[i].is_filled)
         unfilled++;
     }
   if(unfilled >= 3)
      score += 8;
   else
      if(unfilled >= 1)
         score += 5;

   double atr = GetATR(ltf);
   FVG nearest_bullish, nearest_bearish;
   if(FindNearestUnfilledFVG(fvgs, local_price, true, nearest_bullish) &&
      (nearest_bullish.price_top - nearest_bullish.price_bottom) > atr * 0.3)
      score += 5;
   if(FindNearestUnfilledFVG(fvgs, local_price, false, nearest_bearish) &&
      (nearest_bearish.price_top - nearest_bearish.price_bottom) > atr * 0.3)
      score += 5;

   OrderBlock obs[];
   FindOrderBlocksCorrect(ltf, 3, obs, 0, 200);
   UpdateOrderBlockStatus(ltf, obs);
   int active_ob = 0;
   for(int i = 0; i < ArraySize(obs); i++)
     {
      if(!obs[i].is_broken)
         active_ob++;
     }
   if(active_ob >= 2)
      score += 7;
   else
      if(active_ob >= 1)
         score += 4;

   OrderBlock ob_bull, ob_bear;
   if(FindNearestActiveOB(obs, local_price, true, ob_bull) && ob_bull.is_tested && !ob_bull.is_broken)
      score += 5;
   if(FindNearestActiveOB(obs, local_price, false, ob_bear) && ob_bear.is_tested && !ob_bear.is_broken)
      score += 5;

   FVG voids[];
   FindLiquidityVoids(ltf, voids, 0, 200);
   int unfilled_voids = 0;
   for(int i = 0; i < ArraySize(voids); i++)
     {
      if(!voids[i].is_filled)
         unfilled_voids++;
     }
   if(unfilled_voids > 0)
      score += 3;

   return MathMin(25, score);
  }

//+------------------------------------------------------------------+
//| Полная оценка качества сигнала для одного уровня ТФ              |
//+------------------------------------------------------------------+
SignalQuality EvaluateSignalQuality(TFLevel &level, int strength)
  {
   SignalQuality result;
   result.level = level;
   result.score = 0;
   result.is_valid = false;
   result.is_buy = false;
   result.rejection_reason = "";
   double current_price_local = GetCurrentBid();

   result.structure_score = EvaluateStructureQuality(level.htf, strength);
   result.liquidity_score = EvaluateLiquidityQuality(level.htf, level.ltf, strength, current_price_local);
   result.pattern_score = EvaluatePatternQuality(level.ltf, current_price_local);
   result.volatility_score = CalculateVolatilityFitness(level);
   result.score = result.structure_score + result.liquidity_score + result.pattern_score + result.volatility_score;

   LiquidityZone bsl_zones[], ssl_zones[];
   GetAllLiquidityZones(level.htf, strength, bsl_zones, ssl_zones);
   UpdateLiquidityZonesStatus(level.ltf, bsl_zones, 30);
   UpdateLiquidityZonesStatus(level.ltf, ssl_zones, 30);

   bool has_swept_bsl = false, has_swept_ssl = false;
   for(int i = 0; i < ArraySize(bsl_zones); i++)
     {
      if(!bsl_zones[i].is_active)
        {
         has_swept_bsl = true;
         break;
        }
     }
   for(int i = 0; i < ArraySize(ssl_zones); i++)
     {
      if(!ssl_zones[i].is_active)
        {
         has_swept_ssl = true;
         break;
        }
     }

   if(!has_swept_bsl && !has_swept_ssl)
     {
      for(int i = 0; i < ArraySize(bsl_zones); i++)
        {
         if(IsMitigationInProgress(bsl_zones[i], level.ltf))
           {
            has_swept_bsl = true;
            break;
           }
        }
      for(int i = 0; i < ArraySize(ssl_zones); i++)
        {
         if(IsMitigationInProgress(ssl_zones[i], level.ltf))
           {
            has_swept_ssl = true;
            break;
           }
        }
     }

   StructurePoint structure[];
   AnalyzeMarketStructure(level.htf, strength, structure);
   ENUM_MARKET_STRUCTURE trend = GetCurrentTrend(structure);

   if(has_swept_bsl && (trend == STRUCTURE_BEARISH || trend == STRUCTURE_UNDEFINED))
     {
      result.is_buy = false;
      result.is_valid = true;
     }
   else
      if(has_swept_ssl && (trend == STRUCTURE_BULLISH || trend == STRUCTURE_UNDEFINED))
        {
         result.is_buy = true;
         result.is_valid = true;
        }
      else
         if(has_swept_bsl)
           {
            result.is_buy = false;
            result.is_valid = true;
           }
         else
            if(has_swept_ssl)
              {
               result.is_buy = true;
               result.is_valid = true;
              }
            else
              {
               result.is_valid = false;
               result.rejection_reason = "Нет снятия ликвидности";
              }

   return result;
  }

//+------------------------------------------------------------------+
//| Выбор лучшего сигнала из всех уровней ТФ                         |
//+------------------------------------------------------------------+
SignalQuality SelectBestSignal()
  {
   SignalQuality best_signal;
   best_signal.score = -1;
   best_signal.is_valid = false;
   SignalQuality signals[];
   ArrayResize(signals, 0);

   for(int i = 0; i < ArraySize(g_tf_levels); i++)
     {
      if(!g_tf_levels[i].enabled)
         continue;
      SignalQuality sq = EvaluateSignalQuality(g_tf_levels[i], SwingStrength);
      sq.score *= (1.0 + g_tf_levels[i].weight * 0.1);
      int size = ArraySize(signals);
      ArrayResize(signals, size + 1);
      signals[size] = sq;
     }

   for(int i = 0; i < ArraySize(signals) - 1; i++)
      for(int j = i + 1; j < ArraySize(signals); j++)
         if(signals[i].score < signals[j].score)
           {
            SignalQuality tmp = signals[i];
            signals[i] = signals[j];
            signals[j] = tmp;
           }

   for(int i = 0; i < ArraySize(signals); i++)
     {
      if(signals[i].is_valid && signals[i].score >= MinSignalScore)
        {
         best_signal = signals[i];
         return best_signal;
        }
     }
   return best_signal;
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 9 🔹                              |
//|                  РАСЧЁТ SL/TP И МУЛЬТИ-ЦЕЛИ                      |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Поиск противоположной зоны ликвидности для TP                    |
//+------------------------------------------------------------------+
bool FindOppositeLiquidityTarget(LiquidityZone &zones[], double local_price, bool looking_above,
                                 double &target_price, ENUM_TIMEFRAMES tf = 0)
  {
   target_price = 0;
   if(tf == 0)
      tf = AdaptiveTimeframes ? g_tf_levels[1].ltf : FixedLTF;
   double atr = GetATR(tf, 14, 0);
   if(atr <= 0)
      atr = 0.0001;
   double min_distance = DBL_MAX;

   for(int i = 0; i < ArraySize(zones); i++)
     {
      if(!zones[i].is_active)
         continue;
      double distance = looking_above ? zones[i].price_center - local_price : local_price - zones[i].price_center;
      if(distance <= 0)
         continue;
      if(distance > atr * 1.5 && distance < min_distance)
        {
         min_distance = distance;
         target_price = zones[i].price_center;
        }
     }
   return target_price > 0;
  }

//+------------------------------------------------------------------+
//| Поиск целей для SELL сделки (SSL зоны и структурные минимумы)    |
//+------------------------------------------------------------------+
void FindSellTargets(LiquidityZone &ssl_zones[], StructurePoint &structure[], double local_price,
                     LiquidityTarget &targets[], ENUM_TIMEFRAMES tf = 0)
  {
   ArrayResize(targets, 0);
   if(tf == 0)
      tf = AdaptiveTimeframes ? g_tf_levels[1].ltf : FixedLTF;
   double atr = GetATR(tf, 14, 0);
   if(atr <= 0)
      atr = 0.0001;

   struct Candidate { double price; string source; int priority; };
   Candidate candidates[];
   ArrayResize(candidates, 0);

   for(int i = 0; i < ArraySize(ssl_zones); i++)
     {
      if(!ssl_zones[i].is_active || ssl_zones[i].price_center >= local_price)
         continue;
      int size = ArraySize(candidates);
      ArrayResize(candidates, size + 1);
      candidates[size].price = ssl_zones[i].price_center;
      candidates[size].source = "SSL";
      candidates[size].priority = (int)((local_price - ssl_zones[i].price_center) / atr);
     }
   for(int i = 0; i < ArraySize(structure); i++)
     {
      if(structure[i].is_high || structure[i].price >= local_price)
         continue;
      if(!structure[i].is_bos && !structure[i].is_choch)
         continue;
      int size = ArraySize(candidates);
      ArrayResize(candidates, size + 1);
      candidates[size].price = structure[i].price;
      candidates[size].source = "Structure";
      candidates[size].priority = (int)((local_price - structure[i].price) / atr);
     }

   for(int i = 0; i < ArraySize(candidates) - 1; i++)
      for(int j = i + 1; j < ArraySize(candidates); j++)
         if(candidates[i].priority > candidates[j].priority)
           {
            Candidate tmp = candidates[i];
            candidates[i] = candidates[j];
            candidates[j] = tmp;
           }

   Candidate unique[];
   ArrayResize(unique, 0);
   for(int i = 0; i < ArraySize(candidates); i++)
     {
      bool is_duplicate = false;
      for(int j = 0; j < ArraySize(unique); j++)
        {
         if(MathAbs(candidates[i].price - unique[j].price) < atr * 0.3)
           {
            is_duplicate = true;
            break;
           }
        }
      if(!is_duplicate)
        {
         int size = ArraySize(unique);
         ArrayResize(unique, size + 1);
         unique[size] = candidates[i];
        }
     }

   int target_count = MathMin(3, ArraySize(unique));
   for(int t = 0; t < target_count; t++)
     {
      int size = ArraySize(targets);
      ArrayResize(targets, size + 1);
      targets[size].price = unique[t].price;
      targets[size].is_valid = true;
      switch(t)
        {
         case 0:
            targets[size].volume_percent = 0.50;
            break;
         case 1:
            targets[size].volume_percent = 0.30;
            break;
         case 2:
            targets[size].volume_percent = 0.20;
            break;
        }
     }
   if(target_count == 1)
      targets[0].volume_percent = 1.0;
   else
      if(target_count == 2)
        {
         targets[0].volume_percent = 0.60;
         targets[1].volume_percent = 0.40;
        }
  }

//+------------------------------------------------------------------+
//| Поиск целей для BUY сделки (BSL зоны и структурные максимумы)    |
//+------------------------------------------------------------------+
void FindBuyTargets(LiquidityZone &bsl_zones[], StructurePoint &structure[], double local_price,
                    LiquidityTarget &targets[], ENUM_TIMEFRAMES tf = 0)
  {
   ArrayResize(targets, 0);
   if(tf == 0)
      tf = AdaptiveTimeframes ? g_tf_levels[1].ltf : FixedLTF;
   double atr = GetATR(tf, 14, 0);
   if(atr <= 0)
      atr = 0.0001;

   struct Candidate { double price; string source; int priority; };
   Candidate candidates[];
   ArrayResize(candidates, 0);

   for(int i = 0; i < ArraySize(bsl_zones); i++)
     {
      if(!bsl_zones[i].is_active || bsl_zones[i].price_center <= local_price)
         continue;
      int size = ArraySize(candidates);
      ArrayResize(candidates, size + 1);
      candidates[size].price = bsl_zones[i].price_center;
      candidates[size].source = "BSL";
      candidates[size].priority = (int)((bsl_zones[i].price_center - local_price) / atr);
     }
   for(int i = 0; i < ArraySize(structure); i++)
     {
      if(!structure[i].is_high || structure[i].price <= local_price)
         continue;
      if(!structure[i].is_bos && !structure[i].is_choch)
         continue;
      int size = ArraySize(candidates);
      ArrayResize(candidates, size + 1);
      candidates[size].price = structure[i].price;
      candidates[size].source = "Structure";
      candidates[size].priority = (int)((local_price - structure[i].price) / atr);
     }

   for(int i = 0; i < ArraySize(candidates) - 1; i++)
      for(int j = i + 1; j < ArraySize(candidates); j++)
         if(candidates[i].priority > candidates[j].priority)
           {
            Candidate tmp = candidates[i];
            candidates[i] = candidates[j];
            candidates[j] = tmp;
           }

   Candidate unique[];
   ArrayResize(unique, 0);
   for(int i = 0; i < ArraySize(candidates); i++)
     {
      bool is_duplicate = false;
      for(int j = 0; j < ArraySize(unique); j++)
        {
         if(MathAbs(candidates[i].price - unique[j].price) < atr * 0.3)
           {
            is_duplicate = true;
            break;
           }
        }
      if(!is_duplicate)
        {
         int size = ArraySize(unique);
         ArrayResize(unique, size + 1);
         unique[size] = candidates[i];
        }
     }

   int target_count = MathMin(3, ArraySize(unique));
   for(int t = 0; t < target_count; t++)
     {
      int size = ArraySize(targets);
      ArrayResize(targets, size + 1);
      targets[size].price = unique[t].price;
      targets[size].is_valid = true;
      switch(t)
        {
         case 0:
            targets[size].volume_percent = 0.50;
            break;
         case 1:
            targets[size].volume_percent = 0.30;
            break;
         case 2:
            targets[size].volume_percent = 0.20;
            break;
        }
     }
   if(target_count == 1)
      targets[0].volume_percent = 1.0;
   else
      if(target_count == 2)
        {
         targets[0].volume_percent = 0.60;
         targets[1].volume_percent = 0.40;
        }
  }

//+------------------------------------------------------------------+
//| Расчёт уровней для SELL сделки (с учётом OB/FVG)                 |
//+------------------------------------------------------------------+
TradeLevels CalculateSellLevels(LiquidityZone &swept_bsl, OrderBlock &bearish_ob, FVG &bearish_fvg,
                                LiquidityZone &ssl_zones[], double local_price, double local_spread,
                                bool use_ob, ENUM_TIMEFRAMES work_tf, double min_rr_ratio = 2.0)
  {
   TradeLevels levels;
   levels.is_valid = false;
   levels.rejection_reason = "";
   levels.entry_price = local_price;
   double point = GetPoint();
   double atr = GetATR(work_tf, 14, 0);
   if(atr <= 0)
      atr = 0.0001;

   double entry_level = 0;
   if(use_ob && !bearish_ob.is_broken)
     {
      entry_level = bearish_ob.price_bottom - local_spread;
      if(local_price > bearish_ob.price_top + atr * 0.5)
        {
         levels.rejection_reason = "Цена слишком далеко от OB";
         return levels;
        }
     }
   else
      if(!use_ob && !bearish_fvg.is_filled)
        {
         entry_level = bearish_fvg.price_top - local_spread;
         if(local_price > bearish_fvg.price_top + atr * 0.5)
           {
            levels.rejection_reason = "Цена слишком далеко от FVG";
            return levels;
           }
        }
      else
        {
         entry_level = local_price;
        }
   levels.entry_price = entry_level;

   double sl_candidate = 0;
   for(int i = 1; i < 30; i++)
     {
      if(GetHigh(work_tf, i) >= swept_bsl.price_bottom && GetClose(work_tf, i) < swept_bsl.price_bottom)
        {
         sl_candidate = GetHigh(work_tf, i) + atr * 0.2;
         break;
        }
     }
   if(sl_candidate == 0)
     {
      if(use_ob && !bearish_ob.is_broken)
         sl_candidate = bearish_ob.price_top + atr * 0.2;
      else
         if(!bearish_fvg.is_filled)
            sl_candidate = bearish_fvg.price_top + atr * 0.2;
         else
            sl_candidate = entry_level + atr * 1.5;
     }
   if(sl_candidate - entry_level < atr * 0.5)
      sl_candidate = entry_level + atr * 0.5;
   levels.stop_loss = sl_candidate;
   levels.sl_points = (sl_candidate - entry_level) / point;

   double tp_target = 0;
   bool has_target = FindOppositeLiquidityTarget(ssl_zones, local_price, false, tp_target, work_tf);
   levels.take_profit = has_target ? entry_level - (entry_level - tp_target) * 0.8 : entry_level - atr * 3.0;
   levels.tp_points = (entry_level - levels.take_profit) / point;
   levels.risk_reward = levels.sl_points > 0 ? levels.tp_points / levels.sl_points : 0;

   if(levels.risk_reward < min_rr_ratio)
     {
      levels.rejection_reason = "R/R ниже минимума";
      return levels;
     }
   if(levels.tp_points < atr / point * 1.5)
     {
      levels.rejection_reason = "TP слишком близко";
      return levels;
     }
   levels.is_valid = true;
   return levels;
  }

//+------------------------------------------------------------------+
//| Расчёт уровней для BUY сделки (с учётом OB/FVG)                  |
//+------------------------------------------------------------------+
TradeLevels CalculateBuyLevels(LiquidityZone &swept_ssl, OrderBlock &bullish_ob, FVG &bullish_fvg,
                               LiquidityZone &bsl_zones[], double local_price, double local_spread,
                               bool use_ob, ENUM_TIMEFRAMES work_tf, double min_rr_ratio = 2.0)
  {
   TradeLevels levels;
   levels.is_valid = false;
   levels.rejection_reason = "";
   levels.entry_price = local_price;
   double point = GetPoint();
   double atr = GetATR(work_tf, 14, 0);
   if(atr <= 0)
      atr = 0.0001;

   double entry_level = 0;
   if(use_ob && !bullish_ob.is_broken)
     {
      entry_level = bullish_ob.price_top + local_spread;
      if(local_price < bullish_ob.price_bottom - atr * 0.5)
        {
         levels.rejection_reason = "Цена слишком далеко от OB";
         return levels;
        }
     }
   else
      if(!use_ob && !bullish_fvg.is_filled)
        {
         entry_level = bullish_fvg.price_bottom + local_spread;
         if(local_price < bullish_fvg.price_bottom - atr * 0.5)
           {
            levels.rejection_reason = "Цена слишком далеко от FVG";
            return levels;
           }
        }
      else
        {
         entry_level = local_price;
        }
   levels.entry_price = entry_level;

   double sl_candidate = 0;
   for(int i = 1; i < 30; i++)
     {
      if(GetLow(work_tf, i) <= swept_ssl.price_top && GetClose(work_tf, i) > swept_ssl.price_top)
        {
         sl_candidate = GetLow(work_tf, i) - atr * 0.2;
         break;
        }
     }
   if(sl_candidate == 0)
     {
      if(use_ob && !bullish_ob.is_broken)
         sl_candidate = bullish_ob.price_bottom - atr * 0.2;
      else
         if(!bullish_fvg.is_filled)
            sl_candidate = bullish_fvg.price_bottom - atr * 0.2;
         else
            sl_candidate = entry_level - atr * 1.5;
     }
   if(entry_level - sl_candidate < atr * 0.5)
      sl_candidate = entry_level - atr * 0.5;
   levels.stop_loss = sl_candidate;
   levels.sl_points = (entry_level - sl_candidate) / point;

   double tp_target = 0;
   bool has_target = FindOppositeLiquidityTarget(bsl_zones, local_price, true, tp_target, work_tf);
   levels.take_profit = has_target ? entry_level + (tp_target - entry_level) * 0.8 : entry_level + atr * 3.0;
   levels.tp_points = (levels.take_profit - entry_level) / point;
   levels.risk_reward = levels.sl_points > 0 ? levels.tp_points / levels.sl_points : 0;

   if(levels.risk_reward < min_rr_ratio)
     {
      levels.rejection_reason = "R/R ниже минимума";
      return levels;
     }
   if(levels.tp_points < atr / point * 1.5)
     {
      levels.rejection_reason = "TP слишком близко";
      return levels;
     }
   levels.is_valid = true;
   return levels;
  }

//+------------------------------------------------------------------+
//| Адаптивный расчет лота                                           |
//+------------------------------------------------------------------+
double CalculateLotSizeAdaptive(double sl_points)
  {
   double lot_size = 0;
   double min_lot = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_MIN);

// 1. Проверка входных параметров
   if(sl_points <= 0)
     {
      Print("✗ Некорректные параметры: SL=", sl_points);
      return min_lot;
     }

// 2. Получаем баланс
   double account_balance = AccountInfoDouble(ACCOUNT_BALANCE);
   if(account_balance <= 0)
     {
      Print("✗ Баланс счета недоступен");
      return min_lot;
     }

// === ИСПРАВЛЕНО: RiskPercent делим на количество экземпляров ===
   int active_instances = GetActiveSMCInstanceCount();
   if(active_instances < 1)
      active_instances = 1;

   double effective_risk = RiskPercent / active_instances;

// Дневной лимит как ограничение сверху
   if(TotalDailyRisk > 0)
     {
      double daily_risk_per_instance = TotalDailyRisk / active_instances;
      if(daily_risk_per_instance < effective_risk)
         effective_risk = daily_risk_per_instance;
     }

// 3. Получаем tick value
   double tick_value = SymbolInfoDouble(g_symbol, SYMBOL_TRADE_TICK_VALUE);
   double tick_size  = SymbolInfoDouble(g_symbol, SYMBOL_TRADE_TICK_SIZE);

   if(tick_value <= 0 || tick_size <= 0)
     {
      Print("⚠ Tick value = ", tick_value, " для ", g_symbol);
      double contract_size = SymbolInfoDouble(g_symbol, SYMBOL_TRADE_CONTRACT_SIZE);
      tick_value = tick_size * contract_size;
      if(tick_value <= 0)
         tick_value = 1.0;
     }

// 4. Расчет риска в деньгах
   double risk_money = account_balance * effective_risk / 100.0;

// 5. Расчет лота
   lot_size = risk_money / (sl_points * tick_value / tick_size);

// 6. Нормализация
   double max_lot  = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_MAX);
   double lot_step = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_STEP);

   lot_size = MathMax(min_lot, MathMin(max_lot, lot_size));
   lot_size = MathFloor(lot_size / lot_step) * lot_step;

   return lot_size;
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 10 🔹                             |
//|                       ОТКРЫТИЕ ОРДЕРОВ                           |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Универсальная функция открытия позиции (BUY или SELL)             |
//|                                                                   |
//| Автоматически определяет цену: Ask для BUY, Bid для SELL.         |
//| Обрабатывает реквоты (ошибки 138, 136, 135) с повторными попытками.|
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   cmd         — OP_BUY или OP_SELL                               |
//|   volume      — объём позиции                                    |
//|   stoploss    — стоп-лосс (0 = без стопа)                        |
//|   takeprofit  — тейк-профит (0 = без тейка)                     |
//|   comment     — комментарий к ордеру                             |
//|   magic       — Magic Number (один на весь советник)             |
//|   slippage    — максимальное проскальзывание                     |
//|   max_retries — количество повторных попыток при реквоте         |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   ticket > 0 — успешно открыта позиция                           |
//|   -1         — ошибка открытия                                   |
//+------------------------------------------------------------------+
int OpenPosition(int cmd, double volume, double stoploss, double takeprofit,
                 string comment = "", int magic = 0, int slippage = 30, int max_retries = 5)
  {
// Проверка типа ордера
   if(cmd != OP_BUY && cmd != OP_SELL)
     {
      Log("✗ OpenPosition: неверная команда " + IntegerToString(cmd));
      return -1;
     }

// Проверка минимального объёма
   double min_lot = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_MIN);
   if(volume < min_lot)
     {
      Log("✗ OpenPosition: объём " + DoubleToString(volume, 2) +
          " меньше минимального " + DoubleToString(min_lot, 2));
      return -1;
     }

   int retry_count = 0;
   int ticket = -1;

   while(retry_count < max_retries)
     {
      // Обновляем котировки
      RefreshRates();

      // Определяем цену: Ask для BUY, Bid для SELL
      double price = (cmd == OP_BUY) ? GetCurrentAsk() : GetCurrentBid();

      if(price <= 0)
        {
         Log("✗ OpenPosition: не удалось получить цену");
         return -1;
        }

      // Валидация SL/TP для BUY
      if(cmd == OP_BUY)
        {
         if(stoploss > 0 && stoploss >= price)
           {
            Log("⚠ OpenPosition BUY: SL (" + DoubleToString(stoploss, GetDigits()) +
                ") >= цены (" + DoubleToString(price, GetDigits()) + "). SL сброшен.");
            stoploss = 0;
           }
         if(takeprofit > 0 && takeprofit <= price)
           {
            Log("⚠ OpenPosition BUY: TP (" + DoubleToString(takeprofit, GetDigits()) +
                ") <= цены (" + DoubleToString(price, GetDigits()) + "). TP сброшен.");
            takeprofit = 0;
           }
        }
      // Валидация SL/TP для SELL
      else
        {
         if(stoploss > 0 && stoploss <= price)
           {
            Log("⚠ OpenPosition SELL: SL (" + DoubleToString(stoploss, GetDigits()) +
                ") <= цены (" + DoubleToString(price, GetDigits()) + "). SL сброшен.");
            stoploss = 0;
           }
         if(takeprofit > 0 && takeprofit >= price)
           {
            Log("⚠ OpenPosition SELL: TP (" + DoubleToString(takeprofit, GetDigits()) +
                ") >= цены (" + DoubleToString(price, GetDigits()) + "). TP сброшен.");
            takeprofit = 0;
           }
        }

      // Отправляем ордер
      ticket = OrderSend(g_symbol, cmd, volume, price, slippage, stoploss, takeprofit,
                         comment, magic, 0, clrNONE);

      // Успешно
      if(ticket > 0)
        {
         if(retry_count > 0)
           {
            Log("✓ Позиция #" + IntegerToString(ticket) + " открыта после " +
                IntegerToString(retry_count) + " попыток");
           }
         return ticket;
        }

      // Обрабатываем ошибку
      int error = GetLastError();

      switch(error)
        {
         case 138:   // Requote
         case 136:   // Off quotes
         case 135:   // Price changed
            retry_count++;
            Sleep(100 * retry_count);  // 100, 200, 300, 400, 500 мс
            break;

         default:
            Log("✗ OpenPosition: ошибка " + IntegerToString(error) +
                " | " + ErrorDescription(error));
            return -1;
        }
     }

   Log("✗ OpenPosition: не удалось открыть позицию после " +
       IntegerToString(max_retries) + " попыток");
   return -1;
  }

//+------------------------------------------------------------------+
//| Открытие группы SELL ордеров (мульти-цели)                       |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//| Открытие группы SELL ордеров                                      |
//|                                                                   |
//| Создаёт несколько SELL-позиций с разными тейк-профитами.          |
//| Общий объём распределяется согласно volume_percent в каждой цели. |
//|                                                                   |
//| Все ордера внутри группы получают:                                |
//|   • Одинаковый Magic Number                                      |
//|   • Одинаковый stop_loss                                         |
//|   • Комментарий вида "GroupName_TP1", "GroupName_TP2" и т.д.    |
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   targets[]      — массив целей (TP) с весами                    |
//|   stop_loss      — общий стоп-лосс для всех ордеров              |
//|   total_lot_size — суммарный объём на все цели                   |
//|   group_name     — имя группы (для комментария)                  |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   Структура TradeGroup с информацией об открытых ордерах          |
//+------------------------------------------------------------------+
TradeGroup OpenSellGroup(LiquidityTarget &targets[], double stop_loss, double total_lot_size, string group_name)
  {
   TradeGroup group;
   group.total_volume = 0;
   group.order_count = 0;
   group.stop_loss = stop_loss;
   group.entry_price = GetCurrentBid();
   group.is_buy = false;
   group.open_time = TimeCurrent();
   group.group_name = group_name;
   ArrayInitialize(group.tickets, -1);

// Параметры инструмента
   double min_lot  = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_MIN);
   double lot_step = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_STEP);

// Единый Magic Number на все ордера советника
   int magic = GetMagicNumber();

// Проскальзывание из внешних параметров
   int slippage = Slippage;

// Открываем ордер на каждую цель
   for(int t = 0; t < ArraySize(targets); t++)
     {
      // Пропускаем невалидные цели
      if(!targets[t].is_valid)
         continue;

      // Рассчитываем объём для этой цели
      double target_lot = total_lot_size * targets[t].volume_percent;
      target_lot = MathFloor(target_lot / lot_step) * lot_step;

      // Проверяем минимальный объём
      if(target_lot < min_lot)
        {
         Log("⚠ OpenSellGroup: объём цели #" + IntegerToString(t + 1) +
             " (" + DoubleToString(target_lot, 2) + ") меньше минимального. Пропускаем.");
         continue;
        }

      // Комментарий: имя группы + номер цели
      string comment = group_name + "_TP" + IntegerToString(t + 1);

      // Тейк-профит для этой цели
      double tp = targets[t].price;

      // Проверка лимита ДО открытия ордера
      if(group.order_count >= 5)
        {
         Log("⚠ Достигнут лимит ордеров в группе (5). Остальные цели пропущены.");
         break;
        }

      // Открываем позицию
      int ticket = OpenPosition(OP_SELL, target_lot, stop_loss, tp, comment, magic, slippage);

      if(ticket > 0)
        {
         // Сохраняем тикет в группу
         group.tickets[group.order_count] = ticket;
         group.order_count++;
         group.total_volume += target_lot;

         Log("✓ Sell TP" + IntegerToString(t + 1) + " открыт. " +
             "Ticket: #" + IntegerToString(ticket) +
             " | Лот: " + DoubleToString(target_lot, 2) +
             " | TP: " + DoubleToString(tp, GetDigits()) +
             " | SL: " + DoubleToString(stop_loss, GetDigits()));
        }
      else
        {
         Log("✗ Ошибка открытия Sell TP" + IntegerToString(t + 1) +
             " | Лот: " + DoubleToString(target_lot, 2) +
             " | TP: " + DoubleToString(tp, GetDigits()));
        }
     }

// Итоговый лог по группе
   if(group.order_count > 0)
     {
      Log("=== Группа SELL '" + group_name + "' открыта ===" +
          " | Ордеров: " + IntegerToString(group.order_count) +
          " | Общий объём: " + DoubleToString(group.total_volume, 2) +
          " | Цена входа: " + DoubleToString(group.entry_price, GetDigits()) +
          " | Magic: " + IntegerToString(magic));
     }
   else
     {
      Log("✗ Группа SELL '" + group_name + "' не открыта (0 ордеров)");
     }

   return group;
  }

//+------------------------------------------------------------------+
//| Открытие группы BUY ордеров (мульти-цели)                        |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//| Открытие группы BUY ордеров                                       |
//|                                                                   |
//| Создаёт несколько BUY-позиций с разными тейк-профитами.           |
//| Общий объём распределяется согласно volume_percent в каждой цели. |
//|                                                                   |
//| Все ордера внутри группы получают:                                |
//|   • Одинаковый Magic Number                                      |
//|   • Одинаковый stop_loss                                         |
//|   • Комментарий вида "GroupName_TP1", "GroupName_TP2" и т.д.    |
//|                                                                   |
//| ПАРАМЕТРЫ:                                                        |
//|   targets[]      — массив целей (TP) с весами                    |
//|   stop_loss      — общий стоп-лосс для всех ордеров              |
//|   total_lot_size — суммарный объём на все цели                   |
//|   group_name     — имя группы (для комментария)                  |
//|                                                                   |
//| ВОЗВРАТ:                                                          |
//|   Структура TradeGroup с информацией об открытых ордерах          |
//+------------------------------------------------------------------+
TradeGroup OpenBuyGroup(LiquidityTarget &targets[], double stop_loss, double total_lot_size, string group_name)
  {
   TradeGroup group;
   group.total_volume = 0;
   group.order_count = 0;
   group.stop_loss = stop_loss;
   group.entry_price = GetCurrentAsk();
   group.is_buy = true;
   group.open_time = TimeCurrent();
   group.group_name = group_name;
   ArrayInitialize(group.tickets, -1);

// Параметры инструмента
   double min_lot  = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_MIN);
   double lot_step = SymbolInfoDouble(g_symbol, SYMBOL_VOLUME_STEP);

// Единый Magic Number на все ордера советника
   int magic = GetMagicNumber();

// Проскальзывание из внешних параметров
   int slippage = Slippage;

// Открываем ордер на каждую цель
   for(int t = 0; t < ArraySize(targets); t++)
     {
      // Пропускаем невалидные цели
      if(!targets[t].is_valid)
         continue;

      // Рассчитываем объём для этой цели
      double target_lot = total_lot_size * targets[t].volume_percent;
      target_lot = MathFloor(target_lot / lot_step) * lot_step;

      // Проверяем минимальный объём
      if(target_lot < min_lot)
        {
         Log("⚠ OpenBuyGroup: объём цели #" + IntegerToString(t + 1) +
             " (" + DoubleToString(target_lot, 2) + ") меньше минимального. Пропускаем.");
         continue;
        }

      // Комментарий: имя группы + номер цели
      string comment = group_name + "_TP" + IntegerToString(t + 1);

      // Тейк-профит для этой цели
      double tp = targets[t].price;

      // Проверка лимита ДО открытия ордера
      if(group.order_count >= 5)
        {
         Log("⚠ Достигнут лимит ордеров в группе (5). Остальные цели пропущены.");
         break;
        }

      // Открываем позицию
      int ticket = OpenPosition(OP_BUY, target_lot, stop_loss, tp, comment, magic, slippage);

      if(ticket > 0)
        {
         // Сохраняем тикет в группу
         group.tickets[group.order_count] = ticket;
         group.order_count++;
         group.total_volume += target_lot;

         Log("✓ Buy TP" + IntegerToString(t + 1) + " открыт. " +
             "Ticket: #" + IntegerToString(ticket) +
             " | Лот: " + DoubleToString(target_lot, 2) +
             " | TP: " + DoubleToString(tp, GetDigits()) +
             " | SL: " + DoubleToString(stop_loss, GetDigits()));
        }
      else
        {
         Log("✗ Ошибка открытия Buy TP" + IntegerToString(t + 1) +
             " | Лот: " + DoubleToString(target_lot, 2) +
             " | TP: " + DoubleToString(tp, GetDigits()));
        }
     }

// Итоговый лог по группе
   if(group.order_count > 0)
     {
      Log("=== Группа BUY '" + group_name + "' открыта ===" +
          " | Ордеров: " + IntegerToString(group.order_count) +
          " | Общий объём: " + DoubleToString(group.total_volume, 2) +
          " | Цена входа: " + DoubleToString(group.entry_price, GetDigits()) +
          " | Magic: " + IntegerToString(magic));
     }
   else
     {
      Log("✗ Группа BUY '" + group_name + "' не открыта (0 ордеров)");
     }

   return group;
  }

//+------------------------------------------------------------------+
//| Закрытие группы ордеров                                          |
//+------------------------------------------------------------------+
void CloseTradeGroup(TradeGroup &group)
  {
   for(int i = 0; i < group.order_count; i++)
     {
      if(OrderSelect(group.tickets[i], SELECT_BY_TICKET) && OrderCloseTime() == 0)
        {
         double close_price = group.is_buy ? GetCurrentBid() : GetCurrentAsk();
         if(OrderClose(OrderTicket(), OrderLots(), close_price, Slippage, clrNONE))
            Log("Закрыт ордер " + IntegerToString(OrderTicket()));
        }
     }
   group.order_count = 0;
   group.total_volume = 0;
   ArrayInitialize(group.tickets, -1);
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 11 🔹                             |
//|              СОХРАНЕНИЕ И ВОССТАНОВЛЕНИЕ КОНТЕКСТА               |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Сохранение контекста ордеров в бинарный файл                     |
//+------------------------------------------------------------------+
void SaveOrderContextToFile(OrderContext &orders[], int count)
  {
   // Очистка имени символа от запрещённых символов
   string symbol_safe = g_symbol;
   StringReplace(symbol_safe, "#", "_");
   StringReplace(symbol_safe, ":", "_");
   StringReplace(symbol_safe, "/", "_");
   StringReplace(symbol_safe, "\\", "_");
   
   string file_name = "DIZEL_SMT_Orders_" + symbol_safe + ".bin";
   
   int handle = FileOpen(file_name, FILE_BIN | FILE_WRITE);
   if(handle == INVALID_HANDLE)
     {
      Print("Ошибка открытия файла: ", file_name, " код: ", GetLastError());
      return;
     }

   // Запись с проверкой
   if(!FileWriteInteger(handle, count))
     {
      Print("Ошибка записи count");
      FileClose(handle);
      return;
     }

   for(int i = 0; i < count; i++)
     {
      FileWriteInteger(handle, orders[i].ticket);
      FileWriteInteger(handle, orders[i].is_buy ? 1 : 0);
      FileWriteDouble(handle, orders[i].entry_price);
      FileWriteDouble(handle, orders[i].stop_loss);
      FileWriteDouble(handle, orders[i].take_profit);
      FileWriteDouble(handle, orders[i].lot_size);
      FileWriteString(handle, orders[i].comment);
      FileWriteLong(handle, orders[i].open_time);
      FileWriteInteger(handle, orders[i].target_index);
     }

   FileClose(handle);
  }

//+------------------------------------------------------------------+
//| Загрузка контекста ордеров из бинарного файла                    |
//+------------------------------------------------------------------+
int LoadOrderContextFromFile(OrderContext &orders[], int max_count = 10)
  {
   // Безопасное имя файла
   string symbol_safe = g_symbol;
   StringReplace(symbol_safe, "#", "_");
   StringReplace(symbol_safe, ":", "_");
   StringReplace(symbol_safe, "/", "_");
   StringReplace(symbol_safe, "\\", "_");
   
   string file_name = "DIZEL_SMT_Orders_" + symbol_safe + ".bin";
   
   if(!FileIsExist(file_name))
      return 0;
   
   int handle = FileOpen(file_name, FILE_BIN | FILE_READ);
   if(handle == INVALID_HANDLE)
     {
      Print("Ошибка открытия файла для чтения: ", GetLastError());
      return 0;
     }
   
   int count = FileReadInteger(handle);
   
   // Проверка на повреждённый файл
   if(count < 0)
     {
      Print("Некорректный размер данных в файле");
      FileClose(handle);
      return 0;
     }
   
   // Ограничение с предупреждением
   if(count > max_count)
     {
      Print("Предупреждение: прочитано ", count, " ордеров, ограничено до ", max_count);
      count = max_count;
     }
   
   // Критически важно: выделить память под массив
   ArrayResize(orders, count);
   
   for(int i = 0; i < count; i++)
     {
      orders[i].ticket = FileReadInteger(handle);
      orders[i].is_buy = (FileReadInteger(handle) == 1);
      orders[i].entry_price = FileReadDouble(handle);
      orders[i].stop_loss = FileReadDouble(handle);
      orders[i].take_profit = FileReadDouble(handle);
      orders[i].lot_size = FileReadDouble(handle);
      orders[i].comment = FileReadString(handle);
      orders[i].open_time = (datetime)FileReadLong(handle);
      orders[i].target_index = FileReadInteger(handle);
      
      // Лучше не активировать автоматически,
      // либо сохранять/читать это поле из файла
      orders[i].is_active = false;
     }
   
   FileClose(handle);
   return count;
  }

//+------------------------------------------------------------------+
//| Получение безопасного имени файла                                |
//+------------------------------------------------------------------+
string GetOrderContextFileName()
  {
   string symbol_safe = g_symbol;
   StringReplace(symbol_safe, "#", "_");
   StringReplace(symbol_safe, ":", "_");
   StringReplace(symbol_safe, "/", "_");
   StringReplace(symbol_safe, "\\", "_");
   return "DIZEL_SMT_Orders_" + symbol_safe + ".bin";
  }

//+------------------------------------------------------------------+
//| Очистка файла контекста ордеров                                  |
//+------------------------------------------------------------------+
void ClearOrderContextFile()
  {
   string file_name = GetOrderContextFileName();
   
   if(FileIsExist(file_name))
     {
      if(!FileDelete(file_name))
         Print("Ошибка удаления файла: ", file_name, " код: ", GetLastError());
     }
  }

//+------------------------------------------------------------------+
//| Восстановление после перезагрузки терминала                      |
//+------------------------------------------------------------------+
void RecoverAfterRestart()
{
   int loaded = LoadOrderContextFromFile(g_active_orders);
   if(loaded == 0)
   {
      Log("Нет сохранённых ордеров.");
      g_active_order_count = 0;
      return;
   }
   
   Log("Найдено " + IntegerToString(loaded) + " сохранённых ордеров.");

   // Живые ордера сдвигаем в начало массива
   int alive = 0;
   for(int i = 0; i < loaded; i++)
   {
      if(OrderSelect(g_active_orders[i].ticket, SELECT_BY_TICKET) && OrderCloseTime() == 0)
      {
         if(alive != i)
            g_active_orders[alive] = g_active_orders[i];
         g_active_orders[alive].is_active = true;
         alive++;
         Log("Ордер " + IntegerToString(g_active_orders[alive-1].ticket) + " под контролем.");
      }
   }

   g_active_order_count = alive;
   ArrayResize(g_active_orders, alive);  // теперь это безопасно — массив динамический

   if(alive == 0)
   {
      ClearOrderContextFile();
      Log("Живых ордеров не осталось.");
   }
   else
   {
      SaveOrderContextToFile(g_active_orders, alive);
   }
}

//+------------------------------------------------------------------+
//| Мониторинг открытых ордеров (обновление статуса)                 |
//+------------------------------------------------------------------+
void MonitorManagedOrders()
  {
   if(g_active_order_count == 0)
      return;
   int still_alive = 0;
   for(int i = 0; i < g_active_order_count; i++)
     {
      if(!g_active_orders[i].is_active)
         continue;
      if(OrderSelect(g_active_orders[i].ticket, SELECT_BY_TICKET))
        {
         if(OrderCloseTime() != 0)
           {
            g_active_orders[i].is_active = false;
            g_last_close_time = TimeCurrent();
            Log("Ордер " + IntegerToString(g_active_orders[i].ticket) + " закрыт. Profit: " + DoubleToString(OrderProfit(), 2));
           }
         else
            still_alive++;
        }
      else
         g_active_orders[i].is_active = false;
     }
   if(still_alive == 0)
     {
      ClearOrderContextFile();
      g_active_order_count = 0;
     }
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 12 🔹                             |
//|         ФИЛЬТРЫ (СЕССИИ, KILLZONE, ПРОВЕРКА ПОЛЬЗОВАТЕЛЯ)        |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Проверка: находится ли текущее время в Killzone                  |
//| (08:00-10:00 серверного времени, 13:00-16:00)                    |
//+------------------------------------------------------------------+
bool IsInKillzone()
  {
   datetime now = TimeCurrent();  // ✅ Серверное время
   MqlDateTime dt;
   TimeToStruct(now, dt);
   int minutes = dt.hour * 60 + dt.min;

// London Killzone (08:00-10:00)
   if(minutes >= 480 && minutes < 600)
      return true;

// London-New York Overlap (13:00-16:00)
   if(minutes >= 780 && minutes < 960)
      return true;

// Asian Killzone (00:00-02:00) - опционально
   if(EnableAsianKillzone && minutes >= 0 && minutes < 120)
      return true;

// Sydney Killzone (22:00-00:00) - переход через полночь
   if(EnableAsianKillzone && minutes >= 1320)
      return true;

   return false;
  }

//+------------------------------------------------------------------+
//| Проверка: разрешена ли торговля в текущую сессию                 |
//+------------------------------------------------------------------+
bool IsTradingAllowed()
  {
   if(!EnableSessionFilter)
      return true;

// Если включён режим только Killzone
   if(EnableKillzoneOnly)
      return IsInKillzone();

// Иначе проверяем обычные сессии
   for(int i = 0; i < ArraySize(g_sessions); i++)
     {
      if(g_sessions[i].enabled &&
         IsWithinTimeWindow(g_sessions[i].start_hour,
                            g_sessions[i].start_minute,
                            g_sessions[i].end_hour,
                            g_sessions[i].end_minute))
         return true;
     }
   return false;
  }

//+------------------------------------------------------------------+
//| Получение названия текущей торговой сессии                       |
//+------------------------------------------------------------------+
string GetCurrentSessionName()
  {
   if(EnableKillzoneOnly)
      return IsInKillzone() ? "Killzone" : "Closed";

   for(int i = 0; i < ArraySize(g_sessions); i++)
     {
      if(g_sessions[i].enabled &&
         IsWithinTimeWindow(g_sessions[i].start_hour,
                            g_sessions[i].start_minute,
                            g_sessions[i].end_hour,
                            g_sessions[i].end_minute))
         return g_sessions[i].name;
     }
   return "Closed";
  }

//+------------------------------------------------------------------+
//| Проверка: разрешена ли торговля для конкретного символа          |
//+------------------------------------------------------------------+
bool IsSymbolTradingAllowed(string symbol)
  {
   if(!EnableSessionFilter)
      return true;

// Если включён режим Killzone, используем его
   if(EnableKillzoneOnly)
      return IsInKillzone();

   string config = GetSymbolSessionConfig(symbol);

   if(ArraySize(g_sessions) == 0)
      return false;

   bool anyActiveByTime = false;

   for(int i = 0; i < ArraySize(g_sessions); i++)
     {
      if(!g_sessions[i].enabled)
         continue;

      if(IsWithinTimeWindow(g_sessions[i].start_hour, g_sessions[i].start_minute,
                            g_sessions[i].end_hour, g_sessions[i].end_minute))
        {
         anyActiveByTime = true;

         if(config == "Default")
            return true;

         if(config == "Asian" && g_sessions[i].name == "Asian")
            return true;

         if(config == "Asian+London" && (g_sessions[i].name == "Asian" || g_sessions[i].name == "London"))
            return true;

         if(config == "London" && g_sessions[i].name == "London")
            return true;

         if(config == "London+NY" && (g_sessions[i].name == "London" ||
                                      g_sessions[i].name == "NewYork" ||
                                      g_sessions[i].name == "London+NY"))
            return true;
        }
     }

   if(!anyActiveByTime)
      return false;

   return false;
  }

//+------------------------------------------------------------------+
//| Преобразование периода в читаемую строку (M1, H1 и т.д.)         |
//+------------------------------------------------------------------+
string PeriodToString(int period)
  {
   switch(period)
     {
      case PERIOD_M1:
         return "M1";   // вместо case 1
      case PERIOD_M5:
         return "M5";   // вместо case 5
      case PERIOD_M15:
         return "M15";  // вместо case 15
      case PERIOD_M30:
         return "M30";  // вместо case 30
      case PERIOD_H1:
         return "H1";   // вместо case 60
      case PERIOD_H4:
         return "H4";   // вместо case 240
      case PERIOD_D1:
         return "D1";   // вместо case 1440
      case PERIOD_W1:
         return "W1";   // вместо case 10080
      case PERIOD_MN1:
         return "MN";   // вместо case 43200
      default:
         return IntegerToString(period);
     }
  }

//+------------------------------------------------------------------+
//| Проверка активности пользователя (смена символа/ТФ)              |
//+------------------------------------------------------------------+
void CheckUserActivity()
  {
   if(LockSymbol && Symbol() != g_symbol && StringLen(TradeSymbol) == 0)
     {
      static datetime last_warn_symbol = 0;
      if(TimeCurrent() - last_warn_symbol > 300)
        {
         last_warn_symbol = TimeCurrent();
         Log("⚠ График переключен на " + Symbol() + ", советник торгует " + g_symbol);
        }
     }
   if(Period() != g_user_period)
     {
      static datetime last_warn_tf = 0;
      if(TimeCurrent() - last_warn_tf > 300)
        {
         last_warn_tf = TimeCurrent();
         Log("⚠ ТФ графика изменён на " + PeriodToString(Period()));
        }
     }
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 12.5 🔹                           |
//|                         ЗАЩИТА ОТ ГЭПОВ                          |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Получение размера гэпа на указанном ТФ                           |
//+------------------------------------------------------------------+
double GetGapSize(ENUM_TIMEFRAMES tf, int shift = 1)
  {
   double prev_close = GetClose(tf, shift + 1);
   double current_open = GetOpen(tf, shift);
   if(prev_close <= 0 || current_open <= 0)
      return 0;
   return MathAbs(current_open - prev_close);
  }

//+------------------------------------------------------------------+
//| Проверка: был ли недавно гэп на указанном ТФ                     |
//+------------------------------------------------------------------+
bool HasRecentGap(ENUM_TIMEFRAMES tf)
  {
   double prev_close = GetClose(tf, 2);   // Базовая цена
   double current_open = GetOpen(tf, 1);   // Текущая цена

   if(prev_close <= 0 || current_open <= 0)
      return false;

   double gap = MathAbs(current_open - prev_close);
   double gapPercent = (gap / prev_close) * 100.0;  // Базовая цена в знаменателе

   return gapPercent > GetMaxGapSizePercent(tf, 1);
  }

//+------------------------------------------------------------------+
//| Автономный расчёт динамического порога гэпа                      |
//+------------------------------------------------------------------+
double GetMaxGapSizePercent(ENUM_TIMEFRAMES tf, int shift = 1)
  {
   double prev_close = GetClose(tf, shift + 1);
   double current_atr = iATR(_Symbol, tf, 14, shift + 1);

   if(current_atr <= 0 || prev_close <= 0)
      return 0.1;

   double current_volatility = (current_atr / prev_close) * 100.0;
   double avg_volatility = 0;
   int periods = 0;

   for(int i = 1; i <= 50; i++)
     {
      double hist_atr = iATR(_Symbol, tf, 14, shift + 1 + i);
      double hist_close = GetClose(tf, shift + 1 + i);

      if(hist_close > 0 && hist_atr > 0)
        {
         avg_volatility += (hist_atr / hist_close) * 100.0;
         periods++;
        }
     }

   if(periods > 0)
      avg_volatility /= periods;
   else
      avg_volatility = current_volatility;

   double MaxGapSizePercent = MathMax(current_volatility, avg_volatility);

   if(MaxGapSizePercent < 0.05)
      MaxGapSizePercent = 0.05;
   if(MaxGapSizePercent > 5.0)
      MaxGapSizePercent = 5.0;

   return MaxGapSizePercent;
  }

//+------------------------------------------------------------------+
//| Определение направления гэпа (1=бычий, -1=медвежий, 0=нет)       |
//+------------------------------------------------------------------+
int GetGapDirection(ENUM_TIMEFRAMES tf, int shift = 1)
  {
   double prev_close = GetClose(tf, shift + 1);
   double current_open = GetOpen(tf, shift);

   if(prev_close <= 0 || current_open <= 0)
      return 0;

   double gap_percent = (MathAbs(current_open - prev_close) / prev_close) * 100.0;
   double MaxGapSizePercent = GetMaxGapSizePercent(tf, shift);

   if(gap_percent <= MaxGapSizePercent)
      return 0;

   return current_open > prev_close ? 1 : -1;
  }

//+------------------------------------------------------------------+
//| Проверка безопасности входа с учётом гэпов                       |
//+------------------------------------------------------------------+
bool IsEntrySafeFromGaps(bool is_buy, ENUM_TIMEFRAMES tf)
  {
   if(!EnableGapProtection || !CheckGapBeforeEntry)
      return true;
   int gap_dir = GetGapDirection(tf, 1);
   if(gap_dir == 0)
      return true;
   if(is_buy && gap_dir < 0)
     {
      Log("⚠ BUY заблокирован: медвежий гэп");
      return false;
     }
   if(!is_buy && gap_dir > 0)
     {
      Log("⚠ SELL заблокирован: бычий гэп");
      return false;
     }
   return true;
  }

//+------------------------------------------------------------------+
//| Проверка: нужно ли закрывать позиции перед выходными             |
//+------------------------------------------------------------------+
bool ShouldCloseBeforeWeekend()
  {
   if(!CloseBeforeWeekend)
      return false;
   datetime gmt = GetGMTTime();
   MqlDateTime dt;
   TimeToStruct(gmt, dt);
   if(dt.day_of_week == 5 && dt.hour >= WeekendCloseHour)
      return true;
   if(dt.day_of_week == 6 || dt.day_of_week == 0)
      return true;
   return false;
  }

//+------------------------------------------------------------------+
//| Закрытие всех позиций перед выходными                            |
//+------------------------------------------------------------------+
void CloseAllBeforeWeekend()
  {
   if(!ShouldCloseBeforeWeekend() || !HasActiveOrdersSafe())
      return;
   Log("🔴 Закрытие всех позиций перед выходными");
   for(int i = OrdersTotal() - 1; i >= 0; i--)
     {
      if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderSymbol() == g_symbol && IsOurOrder(OrderTicket()))
        {
         double close_price = (OrderType() == OP_BUY) ? GetCurrentBid() : GetCurrentAsk();
         if(OrderClose(OrderTicket(), OrderLots(), close_price, Slippage, clrNONE))
            Log("Закрыт ордер " + IntegerToString(OrderTicket()));
        }
     }
   ClearOrderContextFile();
   g_active_order_count = 0;
   signal_buy = false;
   signal_sell = false;
   g_has_last_signal = false;
  }

//+------------------------------------------------------------------+
//| Проверка виртуального стопа (экстренное закрытие при гэпе)       |
//+------------------------------------------------------------------+
bool CheckVirtualStop()
  {
   if(!EnableGapProtection || !HasActiveOrdersSafe())
      return false;
   for(int i = 0; i < OrdersTotal(); i++)
     {
      if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderSymbol() == g_symbol && IsOurOrder(OrderTicket()))
        {
         double open_price = OrderOpenPrice();
         double current_price_local = (OrderType() == OP_BUY) ? GetCurrentBid() : GetCurrentAsk();
         double move_against = (OrderType() == OP_BUY) ? open_price - current_price_local : current_price_local - open_price;
         ENUM_TIMEFRAMES tf = AdaptiveTimeframes ? g_tf_levels[1].ltf : FixedLTF;
         if((move_against / open_price) * 100.0 > GetMaxGapSizePercent(tf, 1) * 3)
           {
            Log("🚨 ЭКСТРЕННОЕ ЗАКРЫТИЕ! Гэп " + DoubleToString((move_against / open_price) * 100.0, 2) + "%");
            double close_price = (OrderType() == OP_BUY) ? GetCurrentBid() : GetCurrentAsk();
            if(OrderClose(OrderTicket(), OrderLots(), close_price, 100, clrNONE))
               Log("Ордер " + IntegerToString(OrderTicket()) + " закрыт по вирт. стопу");
            ClearOrderContextFile();
            g_active_order_count = 0;
            return true;
           }
        }
     }
   return false;
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК  13 🔹                            |
//|         ОБРАБОТЧИК ЗАКРЫТИЯ СВЕЧЕЙ И ДИСПЕТЧЕР СИГНАЛОВ          |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| Обработчик закрытия свечи на указанном таймфрейме                |
//| Обновляет структуру, зоны ликвидности и паттерны                 |
//+------------------------------------------------------------------+
void OnBarClosed(ENUM_TIMEFRAMES tf)
  {
   Log("Закрылась свеча: " + EnumToString(tf));

   for(int i = 0; i < ArraySize(g_tf_levels); i++)
     {
      if(!g_tf_levels[i].enabled)
         continue;

      if(tf == g_tf_levels[i].ltf)
        {
         if(EnableFVG)
           {
            FindFVGs(g_tf_levels[i].ltf, g_ltf_fvgs, 0, 200);
            UpdateFVGStatus(g_tf_levels[i].ltf, g_ltf_fvgs);
           }
         if(EnableOB)
           {
            FindOrderBlocksCorrect(g_tf_levels[i].ltf, 3, g_ltf_obs, 0, 200);
            UpdateOrderBlockStatus(g_tf_levels[i].ltf, g_ltf_obs);
           }
         UpdateLiquidityZonesStatus(g_tf_levels[i].ltf, g_bsl_zones, 30);
         UpdateLiquidityZonesStatus(g_tf_levels[i].ltf, g_ssl_zones, 30);

         SignalQuality sq = EvaluateSignalQuality(g_tf_levels[i], SwingStrength);
         if(sq.is_valid && sq.score >= MinSignalScore)
           {
            Log("🔔 СИГНАЛ: " + g_tf_levels[i].name + " | " + (sq.is_buy ? "BUY" : "SELL") + " | Score: " + DoubleToString(sq.score, 1));
            if(g_pending_signal_count < 10)
              {
               g_pending_signals[g_pending_signal_count] = sq;
               g_pending_signal_count++;
              }
           }
        }

      if(tf == g_tf_levels[i].htf)
        {
         Log("Обновление структуры на " + g_tf_levels[i].name);
         AnalyzeMarketStructure(g_tf_levels[i].htf, SwingStrength, g_htf_structure);
         GetAllLiquidityZones(g_tf_levels[i].htf, SwingStrength, g_bsl_zones, g_ssl_zones);
        }
     }

   if(!AdaptiveTimeframes)
     {
      if(tf == FixedLTF)
        {
         if(EnableFVG)
           {
            FindFVGs(FixedLTF, g_ltf_fvgs, 0, 200);
            UpdateFVGStatus(FixedLTF, g_ltf_fvgs);
           }
         if(EnableOB)
           {
            FindOrderBlocksCorrect(FixedLTF, 3, g_ltf_obs, 0, 200);
            UpdateOrderBlockStatus(FixedLTF, g_ltf_obs);
           }
         UpdateLiquidityZonesStatus(FixedLTF, g_bsl_zones, 30);
         UpdateLiquidityZonesStatus(FixedLTF, g_ssl_zones, 30);
        }
      if(tf == FixedHTF)
        {
         AnalyzeMarketStructure(FixedHTF, SwingStrength, g_htf_structure);
         GetAllLiquidityZones(FixedHTF, SwingStrength, g_bsl_zones, g_ssl_zones);
        }
     }
  }

//+------------------------------------------------------------------+
//| Исполнение лучшего сигнала из очереди                            |
//| Основная торговая логика: проверка фильтров, расчёт SL/TP, вход  |
//+------------------------------------------------------------------+
void ExecuteBestPendingSignal()
  {
   if(g_pending_signal_count == 0 || HasActiveOrdersSafe())
     {
      g_pending_signal_count = 0;
      return;
     }

   for(int i = 0; i < g_pending_signal_count - 1; i++)
      for(int j = i + 1; j < g_pending_signal_count; j++)
         if(g_pending_signals[i].score < g_pending_signals[j].score)
           {
            SignalQuality tmp = g_pending_signals[i];
            g_pending_signals[i] = g_pending_signals[j];
            g_pending_signals[j] = tmp;
           }

   SignalQuality best = g_pending_signals[0];
   if(!best.is_valid || best.score < MinSignalScore || HasOppositeOrders(best.is_buy))
     {
      g_pending_signal_count = 0;
      return;
     }

   ENUM_TIMEFRAMES exec_htf = AdaptiveTimeframes ? best.level.htf : FixedHTF;
   ENUM_TIMEFRAMES exec_ltf = AdaptiveTimeframes ? best.level.ltf : FixedLTF;

   double current_price_local = GetCurrentBid();
   double current_spread_local = GetCurrentSpread();

   if(!IsEntrySafeFromGaps(best.is_buy, exec_ltf))
     {
      g_pending_signal_count = 0;
      return;
     }
   if(!IsValidZoneForDirection(best.is_buy, exec_ltf))
     {
      Log("Направление не соотв. Premium/Discount");
      g_pending_signal_count = 0;
      return;
     }
   if(!IsSpreadAcceptable())
     {
      Log("Спред слишком высок");
      g_pending_signal_count = 0;
      return;
     }

   LiquidityZone bsl_zones_local[], ssl_zones_local[];
   GetAllLiquidityZones(exec_htf, SwingStrength, bsl_zones_local, ssl_zones_local);
   UpdateLiquidityZonesStatus(exec_ltf, bsl_zones_local, 30);
   UpdateLiquidityZonesStatus(exec_ltf, ssl_zones_local, 30);

   FVG fvgs_local[];
   OrderBlock obs_local[];
   FindFVGs(exec_ltf, fvgs_local, 0, 200);
   UpdateFVGStatus(exec_ltf, fvgs_local);
   FindOrderBlocksCorrect(exec_ltf, 3, obs_local, 0, 200);
   UpdateOrderBlockStatus(exec_ltf, obs_local);

   LiquidityZone swept_zone;
   bool swept_found = false;

   if(best.is_buy)
     {
      // Ищем SSL зону (медвежью) с митигейшн
      swept_found = FindSweptZoneWithMitigation(ssl_zones_local, false, exec_ltf, swept_zone);
     }
   else
     {
      // Ищем BSL зону (бычью) с митигейшн
      swept_found = FindSweptZoneWithMitigation(bsl_zones_local, true, exec_ltf, swept_zone);
     }

   if(!swept_found)
     {
      Log("Не найдена снятая зона с подтверждённым митигейшн");
      g_pending_signal_count = 0;
      return;
     }

   FVG nearest_fvg;
   OrderBlock nearest_ob;
   bool has_fvg = FindNearestUnfilledFVG(fvgs_local, current_price_local, best.is_buy, nearest_fvg);
   bool has_ob = FindNearestActiveOB(obs_local, current_price_local, best.is_buy, nearest_ob);
   bool use_ob = PreferOB_Over_FVG && has_ob;

   TradeLevels levels;
   if(best.is_buy)
      levels = CalculateBuyLevels(swept_zone, nearest_ob, nearest_fvg, bsl_zones_local, current_price_local, current_spread_local, use_ob, exec_ltf, MinRiskRewardRatio);
   else
      levels = CalculateSellLevels(swept_zone, nearest_ob, nearest_fvg, ssl_zones_local, current_price_local, current_spread_local, use_ob, exec_ltf, MinRiskRewardRatio);

   if(!levels.is_valid)
     {
      Log("Уровни невалидны: " + levels.rejection_reason);
      g_pending_signal_count = 0;
      return;
     }

   g_last_swept_zone = swept_zone;
   g_last_active_fvg = nearest_fvg;
   g_last_active_ob = nearest_ob;
   g_last_trade_levels = levels;
   g_has_last_signal = true;

   double lot = CalculateLotSizeAdaptive(levels.sl_points);
   if(!HasEnoughMargin(lot))
     {
      Log("Недостаточно маржи");
      g_pending_signal_count = 0;
      return;
     }

   string group_name = "DIZEL_SMT_" + (best.is_buy ? "Buy" : "Sell") + "_" + TimeToString(TimeCurrent(), TIME_DATE | TIME_MINUTES);

   if(EnableMultiTargets)
     {
      LiquidityTarget targets[];
      if(best.is_buy)
         FindBuyTargets(bsl_zones_local, g_htf_structure, current_price_local, targets, exec_ltf);
      else
         FindSellTargets(ssl_zones_local, g_htf_structure, current_price_local, targets, exec_ltf);

      if(ArraySize(targets) > 0)
        {
         if(ArraySize(targets) >= 1)
            targets[0].volume_percent = Target1Volume;
         if(ArraySize(targets) >= 2)
            targets[1].volume_percent = Target2Volume;
         if(ArraySize(targets) >= 3)
            targets[2].volume_percent = Target3Volume;

         TradeGroup group;
         if(best.is_buy)
            group = OpenBuyGroup(targets, levels.stop_loss, lot, group_name);
         else
            group = OpenSellGroup(targets, levels.stop_loss, lot, group_name);

         if(group.order_count > 0)
           {
            for(int i = 0; i < group.order_count && g_active_order_count < 10; i++)
              {
               g_active_orders[g_active_order_count].ticket = group.tickets[i];
               g_active_orders[g_active_order_count].is_buy = best.is_buy;
               g_active_orders[g_active_order_count].entry_price = levels.entry_price;
               g_active_orders[g_active_order_count].stop_loss = levels.stop_loss;
               g_active_orders[g_active_order_count].take_profit = targets[i].price;
               g_active_orders[g_active_order_count].lot_size = lot * targets[i].volume_percent;
               g_active_orders[g_active_order_count].comment = group_name + "_TP" + IntegerToString(i + 1);
               g_active_orders[g_active_order_count].open_time = TimeCurrent();
               g_active_orders[g_active_order_count].target_index = i;
               g_active_orders[g_active_order_count].is_active = true;
               g_active_order_count++;
              }
            SaveOrderContextToFile(g_active_orders, g_active_order_count);
           }
        }
     }
   else
     {
      int cmd = best.is_buy ? OP_BUY : OP_SELL;
      double sl = levels.stop_loss;
      double tp = levels.take_profit;
      string comment = group_name;
      int magic = GetMagicNumber();

      int ticket = OpenPosition(cmd, lot, sl, tp, comment, magic, Slippage);
      if(ticket > 0 && g_active_order_count < 10)
        {
         g_active_orders[g_active_order_count].ticket = ticket;
         g_active_orders[g_active_order_count].is_buy = best.is_buy;
         g_active_orders[g_active_order_count].entry_price = levels.entry_price;
         g_active_orders[g_active_order_count].stop_loss = levels.stop_loss;
         g_active_orders[g_active_order_count].take_profit = levels.take_profit;
         g_active_orders[g_active_order_count].lot_size = lot;
         g_active_orders[g_active_order_count].comment = group_name;
         g_active_orders[g_active_order_count].open_time = TimeCurrent();
         g_active_orders[g_active_order_count].target_index = -1;
         g_active_orders[g_active_order_count].is_active = true;
         g_active_order_count++;
         SaveOrderContextToFile(g_active_orders, g_active_order_count);
        }
     }

   signal_buy = best.is_buy;
   signal_sell = !best.is_buy;
   g_pending_signal_count = 0;
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 14 🔹                             |
//|                     ИНФОРМАЦИОННАЯ ПАНЕЛЬ                        |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

string g_panel_prefix = "DIZEL_Panel_";
int    g_panel_width_full = 380;
int    g_panel_width_mini = 220;
int    g_panel_row_height = 14;
int    g_panel_padding = 6;
int    g_panel_offset_x = 10;
int    g_panel_offset_y = 30;

color  g_color_bg = C'18,18,18';
color  g_color_border = C'50,50,50';
color  g_color_text = C'200,200,200';
color  g_color_header = C'255,200,50';
color  g_color_bull = C'0,200,100';
color  g_color_bear = C'255,80,80';
color  g_color_neutral = C'150,150,150';
color  g_color_active = C'0,255,100';
color  g_color_inactive = C'255,80,80';

//+------------------------------------------------------------------+
//| Создание текстовой метки на панели                               |
//+------------------------------------------------------------------+
void SetPanelLabel(string name, string text, int x, int y, color clr, int font_size = 9)
  {
   string full_name = g_panel_prefix + name;
   if(ObjectFind(0, full_name) < 0)
     {
      ObjectCreate(0, full_name, OBJ_LABEL, 0, 0, 0);
      ObjectSetInteger(0, full_name, OBJPROP_CORNER, InfoPanelCorner);
      ObjectSetInteger(0, full_name, OBJPROP_XDISTANCE, x);
      ObjectSetInteger(0, full_name, OBJPROP_YDISTANCE, y);
     }
   ObjectSetString(0, full_name, OBJPROP_TEXT, text);
   ObjectSetInteger(0, full_name, OBJPROP_COLOR, clr);
   ObjectSetString(0, full_name, OBJPROP_FONT, "Consolas");
   ObjectSetInteger(0, full_name, OBJPROP_FONTSIZE, font_size);
   ObjectSetInteger(0, full_name, OBJPROP_SELECTABLE, false);
   ObjectSetInteger(0, full_name, OBJPROP_HIDDEN, true);
  }

//+------------------------------------------------------------------+
//| Создание фона панели (прямоугольник)                             |
//+------------------------------------------------------------------+
void CreatePanelBackground(string name, int width, int height, int x, int y)
  {
   string bg_name = g_panel_prefix + name;
   if(ObjectFind(0, bg_name) < 0)
     {
      ObjectCreate(0, bg_name, OBJ_RECTANGLE_LABEL, 0, 0, 0);
      ObjectSetInteger(0, bg_name, OBJPROP_CORNER, InfoPanelCorner);
      ObjectSetInteger(0, bg_name, OBJPROP_XDISTANCE, x);
      ObjectSetInteger(0, bg_name, OBJPROP_YDISTANCE, y);
      ObjectSetInteger(0, bg_name, OBJPROP_XSIZE, width);
      ObjectSetInteger(0, bg_name, OBJPROP_YSIZE, height);
      ObjectSetInteger(0, bg_name, OBJPROP_BGCOLOR, g_color_bg);
      ObjectSetInteger(0, bg_name, OBJPROP_BORDER_COLOR, g_color_border);
      ObjectSetInteger(0, bg_name, OBJPROP_BORDER_TYPE, BORDER_FLAT);
      ObjectSetInteger(0, bg_name, OBJPROP_BACK, false);
      ObjectSetInteger(0, bg_name, OBJPROP_SELECTABLE, false);
      ObjectSetInteger(0, bg_name, OBJPROP_HIDDEN, true);
     }
  }

//+------------------------------------------------------------------+
//| Создание разделительной линии на панели                          |
//+------------------------------------------------------------------+
void CreatePanelSeparator(string name, int y, int width, int x)
  {
   string sep_name = g_panel_prefix + name;
   if(ObjectFind(0, sep_name) < 0)
     {
      ObjectCreate(0, sep_name, OBJ_RECTANGLE_LABEL, 0, 0, 0);
      ObjectSetInteger(0, sep_name, OBJPROP_CORNER, InfoPanelCorner);
      ObjectSetInteger(0, sep_name, OBJPROP_XDISTANCE, x + g_panel_padding);
      ObjectSetInteger(0, sep_name, OBJPROP_YDISTANCE, y);
      ObjectSetInteger(0, sep_name, OBJPROP_XSIZE, width - g_panel_padding * 2);
      ObjectSetInteger(0, sep_name, OBJPROP_YSIZE, 1);
      ObjectSetInteger(0, sep_name, OBJPROP_BGCOLOR, g_color_border);
      ObjectSetInteger(0, sep_name, OBJPROP_BORDER_COLOR, clrNONE);
      ObjectSetInteger(0, sep_name, OBJPROP_BACK, false);
      ObjectSetInteger(0, sep_name, OBJPROP_SELECTABLE, false);
      ObjectSetInteger(0, sep_name, OBJPROP_HIDDEN, true);
     }
  }

//+------------------------------------------------------------------+
//| Очистка всех объектов панели                                     |
//+------------------------------------------------------------------+
void ClearAllPanelObjects()
  {
   ObjectsDeleteAll(0, g_panel_prefix);
  }

//+------------------------------------------------------------------+
//| Отрисовка мини-панели (компактный режим)                         |
//+------------------------------------------------------------------+
void DrawMiniPanel()
  {
   int x = g_panel_offset_x, y = g_panel_offset_y, width = g_panel_width_mini, row = 0;
   CreatePanelBackground("bg", width, g_panel_row_height * 4 + 10, x, y);
   SetPanelLabel("r0", "DIZEL SMT PRO  [" + GetCurrentSessionName() + "]", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_header, 10);
   row++;
   string sig_text = "Сигнал: ";
   color sig_color = g_color_neutral;
   if(signal_buy)
     {
      sig_text += "BUY ▲";
      sig_color = g_color_bull;
     }
   else
      if(signal_sell)
        {
         sig_text += "SELL ▼";
         sig_color = g_color_bear;
        }
      else
        {
         sig_text += "НЕТ ▬";
        }
   SetPanelLabel("r1", sig_text + " | Ордеров: " + IntegerToString(CountOurOrdersSafe()), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, sig_color);
   row++;
   double lot_info = CalculateLotSizeAdaptive(50);
   SetPanelLabel("r2", "Риск: " + DoubleToString(RiskPercent, 1) + "% | Лот: ~" + DoubleToString(lot_info, 2), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_text);
  }

//+------------------------------------------------------------------+
//| Отрисовка полной панели (расширенный режим)                      |
//+------------------------------------------------------------------+
void DrawFullPanel()
  {
   int x = g_panel_offset_x, y = g_panel_offset_y, width = g_panel_width_full, row = 0;
   int total_rows = 26, panel_height = g_panel_row_height * total_rows + 20;
   CreatePanelBackground("bg", width, panel_height, x, y);

   SetPanelLabel("r0", "DIZEL SMT PRO v1.0", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_header, 11);
   row++;
   SetPanelLabel("r1", "Сессия: " + GetCurrentSessionName() + " | " + (AdaptiveTimeframes ? "Адаптивный ТФ" : PeriodToString(FixedHTF) + "/" + PeriodToString(FixedLTF)), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_neutral, 8);
   row++;
   CreatePanelSeparator("s1", y + g_panel_padding + row * g_panel_row_height - 3, width, x);

   ENUM_MARKET_STRUCTURE trend = GetCurrentTrend(g_htf_structure);
   string trend_text = "Не определена";
   color trend_color = g_color_neutral;
   if(trend == STRUCTURE_BULLISH)
     {
      trend_text = "Бычья ▲";
      trend_color = g_color_bull;
     }
   if(trend == STRUCTURE_BEARISH)
     {
      trend_text = "Медвежья ▼";
      trend_color = g_color_bear;
     }
   SetPanelLabel("r2", "СТРУКТУРА: " + trend_text, x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, trend_color);
   row++;

   bool has_bos = false, has_choch = false;
   for(int i = MathMax(0, ArraySize(g_htf_structure) - 20); i < ArraySize(g_htf_structure); i++)
     {
      if(g_htf_structure[i].is_bos)
         has_bos = true;
      if(g_htf_structure[i].is_choch)
         has_choch = true;
     }
   SetPanelLabel("r3", "BOS: " + (has_bos ? "✓" : "✗") + " | CHoCH: " + (has_choch ? "✓" : "✗"), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, has_choch ? g_color_active : g_color_text);
   row++;
   CreatePanelSeparator("s2", y + g_panel_padding + row * g_panel_row_height - 3, width, x);

   int bsl_total = ArraySize(g_bsl_zones), bsl_active = CountActiveZones(g_bsl_zones);
   int ssl_total = ArraySize(g_ssl_zones), ssl_active = CountActiveZones(g_ssl_zones);
   SetPanelLabel("r4", "ЛИКВИДНОСТЬ:", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_header);
   row++;
   SetPanelLabel("r5", "BSL: " + IntegerToString(bsl_total) + " (акт: " + IntegerToString(bsl_active) + " | снято: " + IntegerToString(bsl_total - bsl_active) + ")", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, bsl_active > 0 ? g_color_active : g_color_neutral);
   row++;
   SetPanelLabel("r6", "SSL: " + IntegerToString(ssl_total) + " (акт: " + IntegerToString(ssl_active) + " | снято: " + IntegerToString(ssl_total - ssl_active) + ")", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, ssl_active > 0 ? g_color_active : g_color_neutral);
   row++;
   CreatePanelSeparator("s3", y + g_panel_padding + row * g_panel_row_height - 3, width, x);

   int fvg_total = ArraySize(g_ltf_fvgs), fvg_unfilled = CountUnfilledFVG(g_ltf_fvgs);
   int ob_total = ArraySize(g_ltf_obs), ob_active = CountActiveOB(g_ltf_obs);
   SetPanelLabel("r7", "ПАТТЕРНЫ: FVG " + IntegerToString(fvg_total) + " (незап: " + IntegerToString(fvg_unfilled) + ") | OB " + IntegerToString(ob_total) + " (акт: " + IntegerToString(ob_active) + ")", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, (fvg_unfilled + ob_active) > 0 ? g_color_active : g_color_neutral);
   row++;
   CreatePanelSeparator("s4", y + g_panel_padding + row * g_panel_row_height - 3, width, x);

   if(AdaptiveTimeframes)
     {
      SetPanelLabel("r8", "АДАПТИВНЫЙ ВЫБОР ТФ:", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_header);
      row++;
      for(int i = 0; i < ArraySize(g_tf_levels); i++)
        {
         if(!g_tf_levels[i].enabled)
            continue;
         SignalQuality sq = EvaluateSignalQuality(g_tf_levels[i], SwingStrength);
         string icon = sq.is_valid ? (sq.score >= MinSignalScore ? "✓" : "△") : "✗";
         color ic = sq.is_valid ? (sq.score >= MinSignalScore ? g_color_active : g_color_neutral) : g_color_inactive;
         SetPanelLabel("ad" + IntegerToString(i), g_tf_levels[i].name + " [" + icon + "] " + DoubleToString(sq.score, 1), x + g_panel_padding + 10, y + g_panel_padding + row * g_panel_row_height, ic, 8);
         row++;
        }
      CreatePanelSeparator("s5", y + g_panel_padding + row * g_panel_row_height - 3, width, x);
     }

   string sig_text = "НЕТ ▬";
   color sig_color = g_color_neutral;
   if(signal_buy)
     {
      sig_text = "BUY ▲ | АКТИВЕН";
      sig_color = g_color_bull;
     }
   if(signal_sell)
     {
      sig_text = "SELL ▼ | АКТИВЕН";
      sig_color = g_color_bear;
     }
   SetPanelLabel("r9", "СИГНАЛ: " + sig_text, x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, sig_color);
   row++;

   if(g_has_last_signal && (signal_buy || signal_sell))
     {
      SetPanelLabel("r9a", "Вход: " + DoubleToString(g_last_trade_levels.entry_price, GetDigits()), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_text, 8);
      row++;
      SetPanelLabel("r9b", "SL: " + DoubleToString(g_last_trade_levels.stop_loss, GetDigits()) + " | TP: " + DoubleToString(g_last_trade_levels.take_profit, GetDigits()), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_text, 8);
      row++;
      color rr_color = g_last_trade_levels.risk_reward >= MinRiskRewardRatio ? g_color_active : g_color_inactive;
      SetPanelLabel("r9c", "R/R: 1:" + DoubleToString(g_last_trade_levels.risk_reward, 1), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, rr_color, 8);
      row++;
     }
   CreatePanelSeparator("s6", y + g_panel_padding + row * g_panel_row_height - 3, width, x);

   SetPanelLabel("r10", "ОРДЕРА:", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_header);
   row++;
   int ord_shown = 0;
   for(int i = 0; i < OrdersTotal() && ord_shown < 5; i++)
     {
      if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderSymbol() == g_symbol && IsOurOrder(OrderTicket()))
        {
         double profit = OrderProfit();
         double pips = (OrderType() == OP_BUY) ? (GetCurrentBid() - OrderOpenPrice()) / GetPoint() : (OrderOpenPrice() - GetCurrentAsk()) / GetPoint();
         color oc = profit >= 0 ? g_color_bull : g_color_bear;
         SetPanelLabel("ord" + IntegerToString(ord_shown), "[" + IntegerToString(ord_shown + 1) + "] " + (OrderType() == OP_BUY ? "BUY" : "SELL") + " " + DoubleToString(OrderLots(), 2) + " | TP: " + DoubleToString(OrderTakeProfit(), GetDigits()) + " | " + DoubleToString(pips, 1) + " пп", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, oc, 8);
         row++;
         ord_shown++;
        }
     }
   if(ord_shown == 0)
     {
      SetPanelLabel("r11", "Нет открытых ордеров", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_neutral, 8);
      row++;
     }
   CreatePanelSeparator("s7", y + g_panel_padding + row * g_panel_row_height - 3, width, x);

   int active_pairs = GetActiveSMCInstanceCount();
   SetPanelLabel("r12", "Активные пары: " + IntegerToString(active_pairs) + "/" + IntegerToString(MaxConcurrentPairs), x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_text);
   row++;

   double daily_pnl = 0, weekly_pnl = 0;
   for(int i = 0; i < OrdersHistoryTotal(); i++)
     {
      if(OrderSelect(i, SELECT_BY_POS, MODE_HISTORY) && OrderSymbol() == g_symbol && IsOurOrder(OrderTicket()))
        {
         if(OrderCloseTime() > TimeCurrent() - 86400)
            daily_pnl += OrderProfit() + OrderSwap();
         if(OrderCloseTime() > TimeCurrent() - 604800)
            weekly_pnl += OrderProfit() + OrderSwap();
        }
     }
   double bal = AccountBalance();
   SetPanelLabel("r13", "P&L: День " + DoubleToString(bal > 0 ? (daily_pnl / bal * 100) : 0, 2) + "% | Неделя " + DoubleToString(bal > 0 ? (weekly_pnl / bal * 100) : 0, 2) + "%", x + g_panel_padding, y + g_panel_padding + row * g_panel_row_height, g_color_text);
  }

//+------------------------------------------------------------------+
//| Главная функция отрисовки панели (диспетчер)                     |
//+------------------------------------------------------------------+
void DrawInfoPanel()
  {
   ClearAllPanelObjects();
   if(InfoPanelMode == 0)
      return;
   if(InfoPanelMode == 1)
      DrawMiniPanel();
   else
      DrawFullPanel();
   ChartRedraw();
  }

//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//|                         🔹 БЛОК 15 🔹                             |
//|                       ONTICK, ONTIMER                            |
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+
//+------------------------------------------------------------------+

//+------------------------------------------------------------------+
//| ГЛАВНЫЙ ОБРАБОТЧИК ТИКОВ                                         |
//| Выполняется при каждом новом тике                                |
//+------------------------------------------------------------------+
void OnTick()
  {
   g_last_tick_received = TimeCurrent();
   if(!IsConnectionAlive())
      return;
   if(!g_trackers_initialized)
     {
      InitializeBarTrackers();
      g_trackers_initialized = true;
     }

   CheckUserActivity();
   UpdateHeartbeat();
   current_price = GetCurrentBid();
   current_spread = GetCurrentSpread();

   if(!IsTradingAllowed() || !IsSymbolTradingAllowed(g_symbol))
     {
      static datetime last_panel = 0;
      if(TimeCurrent() - last_panel > 5)
        {
         last_panel = TimeCurrent();
         DrawInfoPanel();
        }
      return;
     }

   MonitorManagedOrders();

// Трейлинг-стоп для активных ордеров
   if(EnableTrailingStop && g_active_order_count > 0)
     {
      ENUM_TIMEFRAMES trail_tf = AdaptiveTimeframes ? g_tf_levels[1].ltf : FixedLTF;

      // Группируем ордера по группам
      for(int i = 0; i < g_active_order_count; i++)
        {
         if(!g_active_orders[i].is_active)
            continue;

         TradeGroup temp_group;
         temp_group.order_count = 1;
         temp_group.tickets[0] = g_active_orders[i].ticket;
         temp_group.is_buy = g_active_orders[i].is_buy;
         temp_group.entry_price = g_active_orders[i].entry_price;
         temp_group.stop_loss = g_active_orders[i].stop_loss;
         temp_group.group_name = g_active_orders[i].comment;

         UpdateTrailingStopForGroup(temp_group, trail_tf);
        }
     }

   if(EnableGapProtection)
     {
      if(CheckVirtualStop())
         return;
      CloseAllBeforeWeekend();
     }

   if(IsInCooldown())
      return;

   bool any_bar_closed = false;
   for(int i = 0; i < ArraySize(g_bar_trackers); i++)
     {
      if(IsNewBarClosed(g_bar_trackers[i].tf, g_bar_trackers[i].last_bar_time))
        {
         any_bar_closed = true;
         OnBarClosed(g_bar_trackers[i].tf);
        }
     }

   if(any_bar_closed)
      ExecuteBestPendingSignal();

   if(!HasActiveOrdersSafe() && (signal_buy || signal_sell))
     {
      signal_buy = false;
      signal_sell = false;
     }

   static datetime last_panel_update = 0;
   if(TimeCurrent() - last_panel_update > 1)
     {
      last_panel_update = TimeCurrent();
      DrawInfoPanel();
     }
  }

//+------------------------------------------------------------------+
//| ПЕРИОДИЧЕСКИЙ ТАЙМЕР (каждые 30 секунд)                          |
//| Проверка целостности ордеров и обновление панели                 |
//+------------------------------------------------------------------+
void OnTimer()
  {
   if(HasActiveOrdersSafe() && g_active_order_count == 0)
     {
      Log("⚠ Обнаружены ордера без контекста. Восстанавливаем...");
      for(int i = 0; i < OrdersTotal() && g_active_order_count < 10; i++)
        {
         if(OrderSelect(i, SELECT_BY_POS, MODE_TRADES) && OrderSymbol() == g_symbol && IsOurOrder(OrderTicket()))
           {
            g_active_orders[g_active_order_count].ticket = OrderTicket();
            g_active_orders[g_active_order_count].is_buy = (OrderType() == OP_BUY);
            g_active_orders[g_active_order_count].entry_price = OrderOpenPrice();
            g_active_orders[g_active_order_count].stop_loss = OrderStopLoss();
            g_active_orders[g_active_order_count].take_profit = OrderTakeProfit();
            g_active_orders[g_active_order_count].lot_size = OrderLots();
            g_active_orders[g_active_order_count].comment = OrderComment();
            g_active_orders[g_active_order_count].open_time = OrderOpenTime();
            g_active_orders[g_active_order_count].target_index = -1;
            g_active_orders[g_active_order_count].is_active = true;
            g_active_order_count++;
           }
        }
      if(g_active_order_count > 0)
         SaveOrderContextToFile(g_active_orders, g_active_order_count);
     }
   if(InfoPanelMode > 0)
      DrawInfoPanel();
  }
//+------------------------------------------------------------------+
//| КОНЕЦ ФАЙЛА DIZEL_SMC_PRO.mq4                                    |
//+------------------------------------------------------------------+
