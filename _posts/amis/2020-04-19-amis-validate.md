---
layout: post
title: "AMis 的验证逻辑"
author: "fedono"
---

- 在 `formItem` 中的方法，也就是在配置 `formItem`  的 `schema` ，配置 `validate` 方法，这时候会在 `src/renderers/Form/Control.tsx` 中进行验证

  ```react
  const formItem = this.model as IFormItemStore;
      if (formItem && validate) {
        let finalValidate = promisify(validate.bind(formItem));
        this.hook2 = function() {
          formItem.clearError('control:valdiate');
          return finalValidate(form.data, formItem.value).then((ret: any) => {
            if ((typeof ret === 'string' || Array.isArray(ret)) && ret) {
              formItem.addError(ret, 'control:valdiate');
            }
          });
        };
        addHook(this.hook2);
      }
  ```

- 在组件中自行写的验证方式

  ```react
  if (control && control.validate && this.model) {
        const formItem = this.model as IFormItemStore;
        let validate = promisify(control.validate.bind(control));
        this.hook = function() {
          formItem.clearError('component:valdiate');
  
          return validate(form.data, formItem.value).then(ret => {
            if ((typeof ret === 'string' || Array.isArray(ret)) && ret) {
              formItem.setError(ret, 'component:valdiate');
            }
          });
        };
        addHook(this.hook);
      }
  ```
  
- 直接使用 `validations` 配置已有的验证方式

  ```react
  const model = (this.model = form.registryItem(name, {
        id,
        type,
        required,
        unique,
        value,
        rules: validations, // 在这里配置
        messages: validationErrors,
        multiple,
        delimiter,
        valueField,
        labelField,
        joinValues,
        extractValue
      }));
  ```

  在 `form/control` 中看到了配置，但是什么时候进行 `rules` 的验证呢？我们看到 `form/Control` 中有个`validate` 方法，在`handleBlur` 中会进行调用，
  
  ```react
  validate() {
      const {formStore: form} = this.props;
  
      if (this.model) {
        if (
          this.model.unique &&
          form.parentStore &&
          form.parentStore.storeType === ComboStore.name
        ) {
          const combo = form.parentStore as IComboStore;
          const group = combo.uniques.get(this.model.name) as IUniqueGroup;
          group.items.forEach(item => item.validate());
        } else {
          this.model.validate(this.hook);
          form
            .getItemsByName(this.model.name)
            .forEach(item => item !== this.model && item.validate());
        }
      }
    }
  ```
  
  这时候我们先不看 `comboStore` ，其他的 `formItem` 都是使用的 `this.model.validate(this.hook); ` 来验证，在`store/formItem` 中，执行验证的方法如下
  
  ```react
  if (hook) {
    yield hook();
  }
  
  addError(
    doValidate(self.value, self.form.data, self.rules, self.messages)
  );
  ```
  
  因为所有非直接写`validation` 的验证方式，如在组件中的`validate` 方法，或者在配置 `schema` 的时候的 `validate` 方法，都会添加到 `hook` 中，所以在 `store/formItem` 中，会执行掉这两个方法，然后在 `doValidate` 中会执行掉配置的 `validation` 方法，来执行 `amis` 内部的验证方法



  