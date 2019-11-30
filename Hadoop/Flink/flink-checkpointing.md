# Flink 의 Checkpointing
* Exactly-Once 또는 At-least-Once 처리를 보장하기 위한 flink 의 장치
* consistent snapshot of
    * The current state of an application
    * The position in an input stream


## job 별로 state backend 지정
```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new FsStateBackend("hdfs://namenode:40010/flink/checkpoints"));
```

StreamTask.executeCheckpointing
-> checkpoint 를 위한 쓰레드풀 생성, AsyncCheckpointRunnable 을 돌림

StreamTask.run
StreamTask.reportCompletedSnapshotStates

TaskStateManagerImpl.reportTaskStateSnapshots

RpcCheckpointResponder.acknowledgeCheckpoint

JobMaster.acknowledgeCheckpoint

JobManager.handleCheckpointMessage

CheckpointCoordinator.receiveAcknowledgeMessage
-> CheckpointCoordinator.completePendingCheckpoint


operatorSnapshotsInProgress??


TaskExecutor.triggerCheckpoint
TaskExecutor.triggerCheckpointBarrier
