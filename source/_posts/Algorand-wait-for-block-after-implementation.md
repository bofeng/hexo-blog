---
title: Algorand wait-for-block-after implementation
typora-root-url: ../../source
date: 2023-03-20 11:57:46
tags: [algorand,golang]
---



Algorand has an API to get status after [waiting for a specified round](https://developer.algorand.org/docs/reference/rest-apis/algod/v2/#get-v2statuswait-for-block-afterround)

I saw it gets used in the `waitForConfirmation` function called something like:

In Golang:
```go
status, err = client.StatusAfterBlock(currentRound).Do(context.Background())
```

In Python:
```python
client.status_after_block(current_round)
```

The implementation for this API is that basically Algorand will block the http request until it gets that specified round, or a timeout (1 minute) happens. The key part of the [source code](https://github.com/algorand/go-algorand/blob/master/daemon/algod/api/server/v2/handlers.go#L324) is like this:

```go
    select {
    case <-v2.Shutdown:
        return internalError(ctx, err, errServiceShuttingDown, v2.Log)
    case <-time.After(1 * time.Minute):
    case <-ledger.Wait(basics.Round(round + 1)):
    }

    // Return status after the wait
    return v2.GetStatus(ctx)
```

## Reference

The `waitForConfirmation` in Python:

```python
def wait_for_confirmation(client, transaction_id, timeout):
    start_round = client.status()["last-round"] + 1
    current_round = start_round

    while current_round < start_round + timeout:
        try:
            pending_txn = client.pending_transaction_info(transaction_id)
        except Exception:
            return 
        if pending_txn.get("confirmed-round", 0) > 0:
            return pending_txn
        elif pending_txn["pool-error"]:  
            raise Exception(
                'pool error: {}'.format(pending_txn["pool-error"]))
        client.status_after_block(current_round)                   
        current_round += 1
    raise Exception(
        'pending tx not found in timeout rounds, timeout value = : {}'.format(timeout))
```

The `waitForConfirmation` in Go:

```go
func waitForConfirmation(txID string, client *algod.Client, timeout uint64) (models.PendingTransactionInfoResponse, error) {
    pt := new(models.PendingTransactionInfoResponse)
    if client == nil || txID == "" || timeout < 0 {
        fmt.Printf("Bad arguments for waitForConfirmation")
        var msg = errors.New("Bad arguments for waitForConfirmation")
        return *pt, msg

    }

    status, err := client.Status().Do(context.Background())
    if err != nil {
        fmt.Printf("error getting algod status: %s\n", err)
        var msg = errors.New(strings.Join([]string{"error getting algod status: "}, err.Error()))
        return *pt, msg
    }
    startRound := status.LastRound + 1
    currentRound := startRound

    for currentRound < (startRound + timeout) {

        *pt, _, err = client.PendingTransactionInformation(txID).Do(context.Background())
        if err != nil {
            fmt.Printf("error getting pending transaction: %s\n", err)
            var msg = errors.New(strings.Join([]string{"error getting pending transaction: "}, err.Error()))
            return *pt, msg
        }
        if pt.ConfirmedRound > 0 {
            fmt.Printf("Transaction "+txID+" confirmed in round %d\n", pt.ConfirmedRound)
            return *pt, nil
        }
        if pt.PoolError != "" {
            fmt.Printf("There was a pool error, then the transaction has been rejected!")
            var msg = errors.New("There was a pool error, then the transaction has been rejected")
            return *pt, msg
        }
        fmt.Printf("waiting for confirmation\n")
        status, err = client.StatusAfterBlock(currentRound).Do(context.Background())
        currentRound++
    }
    msg := errors.New("Tx not found in round range")
    return *pt, msg
}
```

The above code are taken from the "how-to" section in this [page](https://developer.algorand.org/docs/features/transactions/signatures/#single-signatures).
