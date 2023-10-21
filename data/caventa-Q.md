1.

Only allow close market can be called 1 time. Suggest to change WildcatMarket#closeMarket

```
bool public isMarketClosed = false;

function closeMarket() external onlyController nonReentrant {
  require(!isMarketClosed, "Market already closed");
  
  // ... (existing code)
  
  isMarketClosed = true;
  emit MarketClosed(block.timestamp);
}
```