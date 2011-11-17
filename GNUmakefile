LC_ALL = C
mirror = /mnt/mirrors/mandriva/official/2011
v = 0

archs = i586 x86_64
srpms = $(shell cat srpms.list)

cachedir = cache
chrootdir = chroot
chroottardir = chroottars
chroottarsarchdir = $(foreach a,$(archs),$(chroottardir)/$(a))
packagesintmpinchroottar = $(foreach p,$(srpms),$(foreach d,$(chroottarsarchdir),$(d)/tmp/$(p)))
chroottars = $(foreach a,$(archs),$(chroottardir)/$(a).tar)
reportdir = report
reportpackages = $(foreach p,$(addprefix $(reportdir)/,$(srpms)),$(foreach a,$(archs),$(p).$(a)))

srpmsincache = $(addprefix $(cachedir)/,$(srpms))

repos1 = main contrib non-free
repos2 = release updates
repos3 = $(foreach r1,$(repos1),$(foreach r2,$(repos2),$(r1)/$(r2)))
repos  = $(foreach r,$(repos3),$(foreach p,SRPMS $(archat)/media,$(p)/$(r)))

percent = %
archat = $(strip $(foreach a,$(archs),$(findstring $(a),$@)))
ifeq ($(v),0)
	output = &>/dev/null
else
	output =
endif

.SECONDEXPANSION:

all: srpms.list $(chroottardir) $(cachedir) $(reportdir) $(chroottars) $(srpmsincache) $(reportpackages)

$(reportpackages):
	#echo $@

# prereq = chroottars/i586/tmp packagesintmpinchroottar
#$(srpmsincache): $(chroottardir)/$(word 1,$(archs)) $$(chroottardir)/$$(word 1,$$(archs))/tmp/$$(notdir $$@)
$(srpmsincache): $$(chroottardir)/$$(word 1,$$(archs))/tmp/$$(notdir $$@)
	cp $$(ls $(chroottardir)/*/tmp/$(notdir $@)) $@

# get path to src.rpm and copy it into chroottars/i586/tmp
# if path is empty (src.rpm does not exists) then create files
# report/name.of.src.rpm.<arch> containing error message
$(packagesintmpinchroottar):
	p=$$(urpmq --urpmi-root $(chroottardir)/$(archat) \
	--sources $(basename $(notdir $@)) 2>.urpmq.error); \
	if [ -z "$$p" ] ; \
	then sudo touch $(chroottardir)/$(archat)/tmp/$(notdir $@); \
	for i in $(archs); do cat .urpmq.error > $(reportdir)/$(notdir $@).$$i; done; \
	else sudo cp $$p $(chroottardir)/$(archat)/tmp/; fi
	-rm .urpmq.error

$(cachedir) $(reportdir) $(chroottardir):
	mkdir -p $@

clearall: delete.cachedir delete.reportdir delete.chroottardir delete.chrootdir

delete.%:
	-sudo rm -rf $($*)

delete.chrootdir:
	-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys; :
	-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc; :
	-sudo rm -rf $(chrootdir)

$(chroottarsarchdir): 
	for i in $(repos); do \
	sudo urpmi.addmedia --urpmi-root $@ $$(echo $$i | sed 's!/!_!g') \
	$(mirror)/$$i; done $(output)
	sudo urpmi  --no-suggests --excludedocs --no-verify-rpm --auto --root $@ \
	--urpmi-root $@ shadow-utils rpm tar basesystem-minimal rpm-build \
	rpm-mandriva-setup urpmi rsync bzip2 shadow-utils locales-en $(output)
	sudo chroot $@ 'cd /var/lib/rpm && db51_recover -dv' $(output)

$(chroottars): $(chroottarsarchdir)
	sudo tar -cf $@ -C $(foreach d,$^,$(findstring $(d),$@)) .

