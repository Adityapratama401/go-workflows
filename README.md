# Durable workflows using Go

[![Build & Test](https://github.com/cschleiden/go-workflows/actions/workflows/go.yml/badge.svg?branch=main)](https://github.com/cschleiden/go-workflows/actions/workflows/go.yml)

Borrows heavily from [Temporal](https://github.com/temporalio/temporal) (and since it's a fork also [Cadence](https://github.com/uber/cadence)) as well as [DTFx](https://github.com/Azure/durabletask).

See also:
- https://cschleiden.dev/blog/2022-02-13-go-workflows-part1/
- https://cschleiden.dev/blog/2022-05-02-go-workflows-part2/

## Docs

See http://cschleiden.github.io/go-workflows for the current version of the documentation.

## Simple example

### Workflow

Workflows are written in Go code. The only exception is they must not use any of Go's non-deterministic features (`select`, iteration over a `map`, etc.). Inputs and outputs for workflows and activities have to be serializable:

```go
func Workflow1(ctx workflow.Context, input string) error {
	r1, err := workflow.ExecuteActivity[int](ctx, workflow.DefaultActivityOptions, Activity1, 35, 12).Get(ctx)
	if err != nil {
		panic("error getting activity 1 result")
	}

	log.Println("A1 result:", r1)

	r2, err := workflow.ExecuteActivity[int](ctx, workflow.DefaultActivityOptions, Activity2).Get(ctx)
	if err != nil {
		panic("error getting activity 1 result")
	}

	log.Println("A2 result:", r2)

	return nil
}
```

### Activities

Activities can have side-effects and don't have to be deterministic. They will be executed only once and the result is persisted:

```go
func Activity1(ctx context.Context, a, b int) (int, error) {
	return a + b, nil
}

func Activity2(ctx context.Context) (int, error) {
	return 12, nil
}

```

### Worker

The worker is responsible for executing `Workflows` and `Activities`, both need to be registered with it.

```go
func runWorker(ctx context.Context, mb backend.Backend) {
	w := worker.New(mb, nil)

	w.RegisterWorkflow(Workflow1)

	w.RegisterActivity(Activity1)
	w.RegisterActivity(Activity2)

	if err := w.Start(ctx); err != nil {
		panic("could not start worker")
	}
}
```

### Backend

The backend is responsible for persisting the workflow events. Currently there is an in-memory backend implementation for testing, one using [SQLite](http://sqlite.org), one using MySQL, one using PostgreSQL, and one using Redis.

```go
b := sqlite.NewSqliteBackend("simple.sqlite")
```

### Putting it all together

We can start workflows from the same process the worker runs in -- or they can be separate. Here we use the SQLite backend, spawn a single worker (which then executes both `Workflows` and `Activities`), and then start a single instance of our workflow

```go
func main() {
	ctx := context.Background()

	b := sqlite.NewSqliteBackend("simple.sqlite")

	go runWorker(ctx, b)

	c := client.New(b)

	wf, err := c.CreateWorkflowInstance(ctx, client.WorkflowInstanceOptions{
		InstanceID: uuid.NewString(),
	}, Workflow1, "input-for-workflow")
	if err != nil {
		panic("could not start workflow")
	}

	c2 := make(chan os.Signal, 1)
	signal.Notify(c2, os.Interrupt)
	<-c2
}
```
[connections]
nodes=[arch1.test.open.tectum.io:50000, arch2.test.open.tectum.io:50001,arch3.test.open.tectum.io:50002,arch4.test.open.tectum.io:50003,arch5.test.open.tectum.io:50004,arch6.test.open.tectum.io:50005]
jsonplaceholder.typicode.comhttps://www.binancewv.co/en/support/announcement/binance-updates-the-fdusd-zero-trading-fee-promotion-for-regular-and-vip-1-users-3d8514bbc817423eba52fb23483bcbed?utm_source=new_share&amp;ref=CPA_001XLYJ3F8<img width="864" height="443" alt="1000172242" src="https://github.com/user-attachments/assets/26fd6f51-7568-4379-87b0-bddbe1fe27c6" />
<img width="1253" height="832" alt="1000172602" src="https://github.com/user-attachments/assets/50ce78d7-deae-4b26-9ff9-a3ef61d081c3" />
![1000176598](https://github.com/user-attachments/assets/35749710-bd5a-417f-893f-04ec3f7b0df7)
![1000187601](https://github.com/user-attachments/assets/f8ce7261-4c10-435f-926b-6d0e64f8422c)
