//@version=6
strategy("Gold XAUUSD Trading Report v1",
     overlay=true,
     initial_capital=1000,
     currency=currency.USD,
     pyramiding=0,
     commission_type=strategy.commission.percent,
     commission_value=0.0)

// ===== SETTINGS =====
fastEMA = input.int(9, "Fast EMA")
slowEMA = input.int(21, "Slow EMA")
rsiLen  = input.int(14, "RSI Length")

rsiBuy  = input.int(55, "BUY RSI")
rsiSell = input.int(45, "SELL RSI")

atrLen  = input.int(14, "ATR Length")
slATR   = input.float(1.5, "Stop Loss ATR")
tpATR   = input.float(2.5, "Take Profit ATR")

// ===== INDICATORS =====
emaFast = ta.ema(close, fastEMA)
emaSlow = ta.ema(close, slowEMA)
rsiVal  = ta.rsi(close, rsiLen)
atrVal  = ta.atr(atrLen)

// ===== SIGNALS =====
buySignal  = ta.crossover(emaFast, emaSlow) and rsiVal >= rsiBuy
sellSignal = ta.crossunder(emaFast, emaSlow) and rsiVal <= rsiSell

// ===== BUY =====
if buySignal and strategy.position_size == 0
    strategy.entry("BUY", strategy.long)

// ===== SELL =====
if sellSignal and strategy.position_size == 0
    strategy.entry("SELL", strategy.short)

// ===== EXIT LEVELS =====
longSL  = strategy.position_avg_price - atrVal * slATR
longTP  = strategy.position_avg_price + atrVal * tpATR

shortSL = strategy.position_avg_price + atrVal * slATR
shortTP = strategy.position_avg_price - atrVal * tpATR

strategy.exit("BUY EXIT", "BUY", stop=longSL, limit=longTP)
strategy.exit("SELL EXIT", "SELL", stop=shortSL, limit=shortTP)

// ===== CHART =====
plot(emaFast, "Fast EMA", color=color.green, linewidth=2)
plot(emaSlow, "Slow EMA", color=color.red, linewidth=2)

plotshape(buySignal,
     title="BUY",
     style=shape.labelup,
     location=location.belowbar,
     text="BUY",
     color=color.green,
     textcolor=color.white)

plotshape(sellSignal,
     title="SELL",
     style=shape.labeldown,
     location=location.abovebar,
     text="SELL",
     color=color.red,
     textcolor=color.white)

// ===== REPORT DATA =====
totalTrades = strategy.closedtrades
winning     = strategy.wintrades
losing      = strategy.losstrades

winRate = totalTrades > 0 ? (winning / totalTrades) * 100 : 0.0

grossProfit = strategy.grossprofit
grossLoss   = math.abs(strategy.grossloss)

profitFactor = grossLoss > 0 ? grossProfit / grossLoss : na

netProfit = strategy.netprofit

// ===== REPORT TABLE =====
var table report = table.new(position.top_right, 2, 8,
     border_width=1)

if barstate.islast
    table.cell(report, 0, 0, "GOLD REPORT",
         bgcolor=color.blue, text_color=color.white)

    table.cell(report, 1, 0, "XAUUSD",
         bgcolor=color.blue, text_color=color.white)

    table.cell(report, 0, 1, "Total Trades")
    table.cell(report, 1, 1, str.tostring(totalTrades))

    table.cell(report, 0, 2, "Wins")
    table.cell(report, 1, 2, str.tostring(winning))

    table.cell(report, 0, 3, "Losses")
    table.cell(report, 1, 3, str.tostring(losing))

    table.cell(report, 0, 4, "Win Rate")
    table.cell(report, 1, 4, str.tostring(winRate, "#.##") + "%")

    table.cell(report, 0, 5, "Net Profit")
    table.cell(report, 1, 5, "$" + str.tostring(netProfit, "#.##"))

    table.cell(report, 0, 6, "Profit Factor")
    table.cell(report, 1, 6, str.tostring(profitFactor, "#.##"))

    table.cell(report, 0, 7, "RSI")
    table.cell(report, 1, 7, str.tostring(rsiVal, "#.##"))

// ===== ALERTS =====
alertcondition(buySignal,
     title="Gold BUY",
     message="XAUUSD BUY signal")

alertcondition(sellSignal,
     title="Gold SELL",
     message="XAUUSD SELL signal")
