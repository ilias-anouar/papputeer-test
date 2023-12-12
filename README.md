async function repeat(array, bccToProcess, start, check, action, data, methods, resultManager, processStateManager) {
  const processFunction = check ? process : processV;

  if (action === 'compose') {
    if (check) {
      for (let i = 0; i < array[start].length; i++) {
        await resultManager.startNow({ id_seeds: array[start][i].id_seeds, id_process: data.id_process });
        await resultManager.updateState([{ id_seeds: array[start][i].id_seeds, id_process: data.id_process }], "running");
        await processFunction([array[start][i]], [bccToProcess[start][i]], i, { onlyStarted: false }, methods);
      }
    } else {
      await time(3000);
      await processFunction(array[start], bccToProcess[start], start, { onlyStarted: true }, methods);
      if (array.length > start + 1) {
        await repeat(array, bccToProcess, start + 1, check, action, data, methods, resultManager, processStateManager);
      }
    }
  } else {
    if (check) {
      for (let i = 0; i < array[start].length; i++) {
        await resultManager.startNow({ id_seeds: array[start][i].id_seeds, id_process: data.id_process });
        await resultManager.updateState([{ id_seeds: array[start][i].id_seeds, id_process: data.id_process }], "running");
        await processFunction([array[start][i]], start, { onlyStarted: false });
      }
    } else {
      await time(3000);
      await processFunction(array[start], start, { onlyStarted: true });
      if (array.length > start + 1) {
        await repeat(array, bccToProcess, start + 1, check, action, data, methods, resultManager, processStateManager);
      }
    }
  }
}

// Usage
let check = { startingIndexed: toProcess.length >= 2 ? false : true };
await time(3000);
await repeat(toProcess, bccToProcess, 0, check.startingIndexed, actions[0], data, methods, resultManager, processStateManager);
await time(5000);

let status = { waiting, active, finished: 0, failed: 0, id_process: data.id_process };
console.log(status);
processStateManager.addState(status);



# processV

const processV = async (toProcess, start, option) => {
  console.log(seeds);
  await time(3000);

  while (toProcess.length !== 0 && state !== "STOPPED") {
    state = await composeManager.getProcessState(data.id_process);

    if (state === "STOPPED") {
      break;
    }

    for (let i = 0; i < toProcess.length; i++) {
      let seed = toProcess[0];

      if (option.onlyStarted) {
        await startSeedProcessing(seed);
      }

      state = await composeManager.getProcessState(data.id_process);

      if (state === "STOPPED") {
        break;
      }

      await processSeedActions(seed, option);
    }

    updateProcessState();
  }

  handleProcessCompletion();

  async function startSeedProcessing(seed) {
    await resultManager.startNow({ id_seeds: seed.id_seeds, id_process: data.id_process });
    await resultManager.updateState([{ id_seeds: seed.id_seeds, id_process: data.id_process }], "running");
  }

  async function processSeedActions(seed, option) {
    let { actions, subject, pages, c, options, mode } = extractActions(seed);

    console.log(`Actions: ${actions}`);

    let r = '';

    for (let i = 0; i < actions.length; i++) {
      console.log(`${actions[i]} action start`);
      r += await composeManager.processing({
        data: toProcess[0],
        action: actions[i],
        subject,
        pages,
        count: c,
        options,
        entity: data.entity,
        mode,
      });

      if (i < actions.length - 1) {
        r += ', ';
      }
    }

    r = removeTrailingComma(r);

    console.log(r);
    await resultManager.saveFeedback({ feedback: r, id_seeds: toProcess[0].id_seeds, id_process: data.id_process });

    if (r.indexOf('invalid') === -1) {
      await handleSuccess(seed);
    } else {
      await handleFailure(seed);
    }
  }

  function extractActions(seed) {
    let actions, subject, pages, c, options, mode;

    if (
      seed.action.indexOf('click') === -1 &&
      seed.action.indexOf('count') === -1 &&
      seed.action.indexOf('pages') === -1 &&
      seed.action.indexOf('subject') === -1 &&
      seed.action.indexOf('option') === -1
    ) {
      actions = [seed.action];
    } else {
      actions = seed.action.split(',');

      for (let i = 0; i < actions.length; i++) {
        switch (true) {
          case actions[i].indexOf('option') !== -1:
            mode = actions.pop().split(':')[1];
            break;
          case actions[i].indexOf('markAsStarted') !== -1:
            actions.pop();
            options.markAsStarted = true;
            break;
          case actions[i].indexOf('click') !== -1:
            actions.pop();
            options.click = true;
            break;
          case actions[i].indexOf('markAsImportant') !== -1:
            actions.pop();
            options.markAsImportant = true;
            break;
          case actions[i].indexOf('count') !== -1:
            c = actions.pop().split(':')[1];
            break;
          case actions[i].indexOf('pages') !== -1:
            pages = parseInt(actions.pop().split(':')[1]);
            break;
          case actions[i].indexOf('subject') !== -1:
            subject = actions.pop().split(':')[1];
            break;
        }
      }
    }

    return { actions, subject, pages, c, options, mode };
  }

  function removeTrailingComma(str) {
    const array = str.split(', ');
    array.pop();
    return array.join(', ');
  }

  async function handleSuccess(seed) {
    success++;

    const end_in = new Date();
    const result = {
      id_seeds: seed.id_seeds,
      end_in,
      id_process: data.id_process,
    };

    await Promise.all([
      resultManager.updateState([{ id_seeds: seed.id_seeds, id_process: data.id_process }], "finished"),
      resultManager.endNow(result),
    ]);

    toProcess.shift();

    console.log(seeds.length);
    if (toProcess.length < active && state !== "STOPPED" && seeds.length !== 0) {
      console.log('The indexed seed: ' + seeds[0].id_seeds);
      toProcess.push(seeds[0]);
      seeds.splice(seeds.indexOf(seeds[0]), 1);
      count++;

      updateProcessState();
    }
  }

  async function handleFailure(seed) {
    failed++;

    const end_in = new Date();
    const result = {
      id_seeds: seed.id_seeds,
      end_in,
      id_process: data.id_process,
    };

    await Promise.all([
      resultManager.updateState([{ id_seeds: seed.id_seeds, id_process: data.id_process }], "failed"),
      resultManager.endNow(result),
    ]);

    toProcess.shift();
    state = await composeManager.getProcessState(data.id_process);

    if (state === "STOPPED") {
      return;
    }

    if (toProcess.length < active && count < length && state !== "STOPPED" && seeds.length !== 0) {
      console.log('The indexed seed: ' + seeds[0 + start].id_seeds);
      toProcess.push(seeds[0 + start]);
      seeds.splice(seeds.indexOf(seeds[0 + start]), 1);
      count++;

      updateProcessState();
    }
  }

  function updateProcessState() {
    const w = seeds.length + 3;
    const status = { waiting: Math.max(0, w), active: toProcess.length, finished: success, failed, id_process: data.id_process };
    processStateManager.updateState(status);
  }

  function handleProcessCompletion() {
    let w = seeds.length + 3;

    if (w <= 0) {
      let status = { waiting: 0, active: toProcess.length, finished: success, failed, id_process: data.id_process };
      processStateManager.updateState(status);
    } else {
      let status = { waiting: w, active: toProcess.length, finished: success, failed, id_process: data.id_process };
      processStateManager.updateState(status);
    }

    state = await composeManager.getProcessState(data.id_process);

    if (state === "STOPPED") {
      return;
    }

    if (toProcess.length === 0) {
      let status = { waiting: 0, active: 0, finished: success, failed, id_process: data.id_process };
      await processStateManager.updateState(status);
      composeManager.finishedProcess({ id_process: data.id_process, status: `FINISHED` });
      console.log(`Process with id: ${data.id_process} finished at ${new Date().toLocaleString()}`);
      sendToAll(clients, 'reload');
    }
  }
};


# process const process = async (toProcess, bccToProcess, start, option, methods) => {
  console.log(bccToProcess);
  await time(3000);

  const processSeed = async (seed) => {
    if (option.onlyStarted) {
      await resultManager.startNow({ id_seeds: seed.id_seeds, id_process: data.id_process });
      await resultManager.updateState([{ id_seeds: seed.id_seeds, id_process: data.id_process }], "running");
    }

    let r = '';
    for (let j = 0; j < actions.length; j++) {
      r += await composeManager.processing({
        data: seed,
        action: actions[j],
        subject: subject,
        to: to,
        offer: seed.offer,
        bcc: bccToProcess[0],
        entity: data.entity,
        mode: 'Cookies'
      });

      if (bccToProcess[0] != undefined) {
        bccCount = bccCount + bccToProcess[0].length;
        await composeManager.saveCounter({ counter: bccCount, id_process: data.id_process });
        sendToAll(clients, 'reload');
      }

      if (j < actions.length) {
        r += ', ';
      }
    }

    let array = r.split(', ');
    array.pop();
    r = array.join(', ');

    await resultManager.saveFeedback({ feedback: r, id_seeds: seed.id_seeds, id_process: data.id_process });

    if (r.indexOf('invalid') == -1 && r.indexOf('detected') == -1 && r.indexOf('noData') == -1) {
      success++;
      let end_in = new Date();
      let result;
      await Promise.all([
        await resultManager.updateState([{ id_seeds: seed.id_seeds, id_process: data.id_process }], "finished"),
        result = { id_seeds: seed.id_seeds, end_in: end_in, id_process: data.id_process },
        await resultManager.endNow(result)
      ]);

      bccToProcess.shift();
      toProcess.shift();

      if (toProcess.length < active && state != "STOPPED" && seeds.length != 0 && bccResult.length != 0 && bccResult[0 + start] != undefined) {
        console.log('the indexed seed : ' + seeds[0].id_seeds);
        toProcess.push(seeds[0]);
        bccToProcess.push(bccResult[0 + start]);
        seeds.splice(seeds.indexOf(seeds[0]), 1);
        count++;

        let waiting = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "waiting" });
        let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
        let w = waiting.length;
        let status = { waiting: w, active: running.length, finished: success, failed: failed, id_process: data.id_process };
        processStateManager.updateState(status);
      }

      if (seeds.length == 0 && bccToProcess.length == 0 && bccResult[0 + start] != undefined && bccResult.length != 0 && Origins.length != 0) {
        seeds = [...Origins];
        await time(2000);
        await resultManager.updateState([{ id_seeds: seeds[0].id_seeds, id_process: data.id_process }], "running");
        toProcess.push(seeds[0]);
        seeds.splice(seeds.indexOf(seeds[0]), 1);
        bccToProcess.push(bccResult[0 + start]);
        bccResult.splice(bccResult.indexOf(bccResult[0 + start]), 1);
      }
    } else {
      failed++;
      let end_in = new Date();
      let result;
      await Promise.all([
        await resultManager.updateState([{ id_seeds: seed.id_seeds, id_process: data.id_process }], "failed"),
        result = { id_seeds: seed.id_seeds, end_in: end_in, id_process: data.id_process },
        await resultManager.endNow(result)
      ]);

      Origins.splice(Origins.indexOf(seed), 1);
      bccToProcess.shift();
      toProcess.shift();

      state = await composeManager.getProcessState(data.id_process);
      if (state == "STOPPED") {
        break;
      }

      if (toProcess.length < active && state != "STOPPED" && seeds.length != 0 && bccResult.length != 0 && bccResult[0 + start] != undefined) {
        console.log('the indexed seed : ' + seeds[0].id_seeds);
        toProcess.push(seeds[0]);
        await resultManager.updateState([{ id_seeds: seeds[0].id_seeds, id_process: data.id_process }], "running");
        seeds.splice(seeds.indexOf(seeds[0]), 1);

        if (bccResult[0 + start] != undefined) {
          bccToProcess.push(bccResult[0 + start]);
          bccResult.splice(bccResult.indexOf(bccResult[0 + start]), 1);
        }

        count++;

        let waiting = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "waiting" });
        let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
        let w = waiting.length;
        let status = { waiting: w, active: running.length, finished: success, failed: failed, id_process: data.id_process };
        processStateManager.updateState(status);
      }

      if (seeds.length == 0 && bccToProcess.length == 0 && bccResult[0 + start] != undefined && bccResult.length != 0 && Origins.length != 0) {
        seeds = [...Origins];
        await time(2000);
        await resultManager.updateState([{ id_seeds: seeds[0].id_seeds, id_process: data.id_process }], "running");
        toProcess.push(seeds[0]);
        seeds.splice(seeds.indexOf(seeds[0]), 1);
        bccToProcess.push(bccResult[0 + start]);
        bccResult.splice(bccResult.indexOf(bccResult[0 + start]), 1);
      }
    }
  };

  switch (methods.fixedLimit) {
    case true:
      console.log('true');
      while (toProcess.length != 0 && state != "STOPPED") {
        state = await composeManager.getProcessState(data.id_process);
        if (state == "STOPPED") {
          break;
        }

        for (let i = 0; i < toProcess.length; i++) {
          await processSeed(toProcess[0]);
        }

        let waiting = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "waiting" });
        let w = waiting.length;

        if (w <= 0) {
          let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
          let status = { waiting: 0, active: running.length, finished: success, failed: failed, id_process: data.id_process };
          processStateManager.updateState(status);
        } else {
          let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
          let status = { waiting: w, active: running.length, finished: success, failed: failed, id_process: data.id_process };
          processStateManager.updateState(status);
        }

        state = await composeManager.getProcessState(data.id_process);

        if (state == "STOPPED") {
          break;
        }

        if (toProcess.length == 0) {
          let status = { waiting: 0, active: 0, finished: success, failed: failed, id_process: data.id_process };
          await processStateManager.updateState(status);
          composeManager.finishedProcess({ id_process: data.id_process, status: `FINISHED` });
          console.log(`process with id : ${data.id_process} Finished At ${new Date().toLocaleString()}`);
          sendToAll(clients, 'reload');
        }
      }
      break;
    case 'none':
      console.log('none');
      while (toProcess.length != 0 && state != "STOPPED") {
        state = await composeManager.getProcessState(data.id_process);
        if (state == "STOPPED") {
          break;
        }

        for (let i = 0; i < toProcess.length; i++) {
          await processSeed(toProcess[0]);
        }

        let waiting = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "waiting" });
        let w = waiting.length;

        if (w <= 0) {
          let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
          let status = { waiting: 0, active: running.length, finished: success, failed: failed, id_process: data.id_process };
          processStateManager.updateState(status);
        } else {
          let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
          let status = { waiting: w, active: running.length, finished: success, failed: failed, id_process: data.id_process };
          processStateManager.updateState(status);
        }

        state = await composeManager.getProcessState(data.id_process);

        if (state == "STOPPED") {
          break;
        }

        if (toProcess.length == 0) {
          let status = { waiting: 0, active: 0, finished: success, failed: failed, id_process: data.id_process };
          await processStateManager.updateState(status);
          composeManager.finishedProcess({ id_process: data.id_process, status: `FINISHED` });
          console.log(`process with id : ${data.id_process} Finished At ${new Date().toLocaleString()}`);
          sendToAll(clients, 'reload');
        }
      }
      break;
    default:
      console.log('default');
      while (bccToProcess.length != 0 && state != "STOPPED") {
        state = await composeManager.getProcessState(data.id_process);
        if (state == "STOPPED") {
          break;
        }

        for (let i = 0; i < toProcess.length; i++) {
          if (bccToProcess.length == 0) {
            break;
          }

          if (bccToProcess[0] == undefined) {
            bccToProcess.shift();
            break;
          }

          await processSeed(toProcess[0]);
        }

        let waiting = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "waiting" });
        let w = waiting.length;

        if (w <= 0) {
          let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
          let status = { waiting: 0, active: running.length, finished: success, failed: failed, id_process: data.id_process };
          processStateManager.updateState(status);
        } else {
          let running = await composeManager.getAllProcessSeedsByState({ id_process: data.id_process, status: "running" });
          let status = { waiting: w, active: running.length, finished: success, failed: failed, id_process: data.id_process };
          processStateManager.updateState(status);
        }

        state = await composeManager.getProcessState(data.id_process);

        if (state == "STOPPED") {
          break;
        }

        if (bccToProcess.length == 0 && toProcess.length == 0 && bccResult.length == 0) {
          let status = { waiting: 0, active: 0, finished: success, failed: failed, id_process: data.id_process };
          await processStateManager.updateState(status);
          composeManager.finishedProcess({ id_process: data.id_process, status: `FINISHED` });
          console.log(`process with id : ${data.id_process} Finished At ${new Date().toLocaleString()}`);
          sendToAll(clients, 'reload');
        }

        if (Origins.length == 0) {
          let status = { waiting: 0, active: 0, finished: success, failed: failed, id_process: data.id_process };
          await processStateManager.updateState(status);
          composeManager.finishedProcess({ id_process: data.id_process, status: `FINISHED` });
          console.log(`process with id : ${data.id_process} Finished At ${new Date().toLocaleString()}`);
          sendToAll(clients, 'reload');
        }
      }
      break;
  }
};

